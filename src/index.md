---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 2rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

</style>

<div class="hero">
  <h1>Wizualizacja sprzedaży</h1>
  <h2>Używamy do tego celu pliku products.json</h2>
</div>

<div class="grid grid-cols-2" style="grid-auto-rows: 504px;">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Sales by Category",
      width,
      height: 400,
      color: {
        legend: false
      },
      x: {
        label: "Category",
        domain: salesByCategoryArray.map(d => d.category),
        tickSize: 0,
        labelOffset: 15,
        tickRotate: -45
      },
      y: {
        label: "Sales",
        grid: true
      },
      marks: [
        Plot.barY(salesByCategoryArray, {x: "category", y: "sales", fill: d => d.category}),
        Plot.text(salesByCategoryArray, {x: "category", y: "sales", dy: -10, text: d => d.sales.toFixed(2), fill: "black"})
      ],
      style: {
        fontSize: "12px",
        fontFamily: "sans-serif"
      }
    }))
  }</div>
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Scatter Plot of Sales vs. Profit",
      width,
      height: 400,
      x: {
        label: "Sales",
        grid: true
      },
      y: {
        label: "Profit",
        grid: true
      },
      marks: [
        Plot.dot(products, {x: "Sales", y: "Profit", stroke: "Category", fill: "Category", r: 3, title: d => `${d.Category}: ${d.Sales.toFixed(2)} sales, ${d.Profit.toFixed(2)} profit`})
      ],
      style: {
        fontSize: "12px",
        fontFamily: "sans-serif"
      }
    }))
  }</div>
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Bar Chart of Sales by Region",
      width,
      height: 400,
      color: {
        legend: false
      },
      x: {
        label: "Region",
        domain: salesByRegionArray.map(d => d.region),
        tickSize: 0,
        labelOffset: 15,
        tickRotate: -45
      },
      y: {
        label: "Sales",
        grid: true
      },
      marks: [
        Plot.barY(salesByRegionArray, {x: "region", y: "sales", fill: d => d.region}),
        Plot.text(salesByRegionArray, {x: "region", y: "sales", dy: -10, text: d => d.sales.toFixed(2), fill: "black"})
      ],
      style: {
        fontSize: "12px",
        fontFamily: "sans-serif"
      }
    }))
  }</div>
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Histogram of Sales",
      width,
      height: 400,
      x: {
        label: "Sales",
        grid: true
      },
      y: {
        label: "Count",
        grid: true
      },
      marks: [
        Plot.rectY(products, Plot.binX({y: "count"}, {x: "Sales", thresholds: 20})),
        Plot.ruleY([0])
      ],
      style: {
        fontSize: "12px",
        fontFamily: "sans-serif"
      }
    }))
  }</div>
</div>

<div>
  <div id="pie-chart"></div>
</div>

```js

// Load the products.json file
const products = await FileAttachment("products.json").json();

// Ensure dates are parsed correctly
products.forEach(product => {
  if (typeof product.OrderDate === 'string') {
    product.OrderDate = new Date(product.OrderDate);
  }
});

// Summarize sales by category
const salesByCategory = products.reduce((acc, product) => {
  acc[product.Category] = (acc[product.Category] || 0) + product.Sales;
  return acc;
}, {});

// Summarize sales by region
const salesByRegion = products.reduce((acc, product) => {
  acc[product.Region] = (acc[product.Region] || 0) + product.Sales;
  return acc;
}, {});

// Summarize profit by category
const profitByCategory = products.reduce((acc, product) => {
  acc[product.Category] = (acc[product.Category] || 0) + product.Profit;
  return acc;
}, {});

// Convert to an array format suitable for Plot
const salesByCategoryArray = Object.keys(salesByCategory).map(key => ({
  category: key,
  sales: salesByCategory[key]
}));

const salesByRegionArray = Object.keys(salesByRegion).map(key => ({
  region: key,
  sales: salesByRegion[key]
}));

const profitByCategoryArray = Object.keys(profitByCategory).map(key => ({
  category: key,
  profit: profitByCategory[key]
}));

// Summarize sales over time by region
const salesOverTimeByRegion = products.reduce((acc, product) => {
  const date = product.OrderDate?.toISOString().split('T')[0];
  if (!acc[date]) {
    acc[date] = {};
  }
  if (!acc[date][product.Region]) {
    acc[date][product.Region] = 0;
  }
  acc[date][product.Region] += product.Sales;
  return acc;
}, {});

const salesOverTimeByRegionArray = Object.keys(salesOverTimeByRegion).flatMap(date => 
  Object.keys(salesOverTimeByRegion[date]).map(region => ({
    date: new Date(date),
    region: region,
    sales: salesOverTimeByRegion[date][region]
  }))
);

// For debugging
salesByCategoryArray;
salesByRegionArray;
profitByCategoryArray;
salesOverTimeByRegionArray;

const pie = d3.pie().value(d => d.profit);
const arcs = pie(profitByCategoryArray);

d3.json("products.json").then(function(data) {
  const width = 500;
  const height = 500;
  const margin = 40;
  const radius = Math.min(width, height) / 2 - margin;

  const svg = d3.select("#pie-chart")
    .append("svg")
    .attr("width", width)
    .attr("height", height)
    .append("g")
    .attr("transform", `translate(${width / 2},${height / 2})`);

  const color = d3.scaleOrdinal()
    .domain(data.map(d => d.Country))
    .range(d3.schemeCategory10);

  const pie = d3.pie()
    .value(d => d.Sales);

  const data_ready = pie(data);

  const arc = d3.arc()
    .innerRadius(0)
    .outerRadius(radius);

  svg.selectAll('pieces')
    .data(data_ready)
    .enter()
    .append('path')
    .attr('d', arc)
    .attr('fill', d => color(d.data.Country))
    .attr("stroke", "white")
    .style("stroke-width", "2px")
    .style("opacity", 0.7);

  svg.selectAll('pieces')
    .data(data_ready)
    .enter()
    .append('text')
    .text(d => d.data.Country)
    .attr("transform", d => `translate(${arc.centroid(d)})`)
    .style("text-anchor", "middle")
    .style("font-size", 12);

});