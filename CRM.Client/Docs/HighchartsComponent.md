# Highcharts Component

## Overview

The Highcharts Blazor component provides a wrapper around the Highcharts JavaScript library
for rendering charts in your Blazor pages. The component handles loading the Highcharts
library, detecting data changes, and re-rendering charts when data updates.

To use this library you must have the required Highcharts JavaScript library files loaded
in your application. The component automatically loads these from the Highcharts CDN:

- highcharts.js
- exporting.js
- export-data.js
- drilldown.js
- accessibility.js

## Files

| File | Purpose |
|------|---------|
| Highcharts.razor | The Blazor component |
| Highcharts.razor.js | JavaScript interop for Highcharts library |
| HighchartsTest.razor | Test page demonstrating all chart types |

## Chart Types

The component supports six chart types through the `ChartTypes` enum:

| Type | Description |
|------|-------------|
| Column | Grouped bar chart with string categories on x-axis |
| Pie | Simple pie chart showing distribution |
| LineTimeSeries | Zoomable line chart with datetime x-axis |
| ColumnTimeSeries | Zoomable bar chart with datetime x-axis |
| AreaTotals | Gradient-filled area chart for cumulative data |
| PieWithDrilldown | Pie chart with click-through detail breakdown |

## Data Models

The component includes nested classes for passing data to charts.

### SeriesData

Used for simple pie charts.

```csharp
public class SeriesData
{
    public string name { get; set; } = "";
    public decimal data { get; set; }
    public string tooltip { get; set; } = "";
}
```

### SeriesDataArray

Used for categorical column charts.

```csharp
public class SeriesDataArray
{
    public string name { get; set; } = "";
    public decimal[] data { get; set; } = [];
}
```

### SeriesDataXYArray

Used for time-series charts (Line, Column, Area). Each data point is an array
containing a Unix timestamp in milliseconds and a value.

```csharp
public class SeriesDataXYArray
{
    public string name { get; set; } = "";
    public long[][] data { get; set; } = [];  // [[timestamp_ms, value], ...]
}
```

### DrilldownSeries

Used for pie charts with drilldown capability. The `id` property must match
the `name` property of the corresponding root SeriesData item.

```csharp
public class DrilldownSeries
{
    public string id { get; set; } = "";
    public DrilldownPoint[] data { get; set; } = [];
}
```

### DrilldownPoint

Individual points within a drilldown series.

```csharp
public class DrilldownPoint
{
    public string name { get; set; } = "";
    public decimal y { get; set; }
}
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| ChartType | ChartTypes? | The type of chart to render |
| ElementId | string? | The id of the target div element |
| ChartTitle | string? | Title displayed at top of chart |
| ChartSubtitle | string? | Subtitle displayed below title |
| yAxisText | string? | Label for the y-axis |
| PreloadResources | bool | Set to true to preload the Highcharts library without rendering |
| ShowZoomHint | bool | Show "drag to zoom" hint in subtitle (default: true) |
| SeriesCategories | string[]? | X-axis labels for categorical charts |
| SeriesDataItems | SeriesData[]? | Data for Pie charts |
| SeriesDataArrayItems | SeriesDataArray[]? | Data for Column charts |
| SeriesDataXYArrayItems | SeriesDataXYArray[]? | Data for time-series charts |
| PieRootItems | SeriesData[]? | Root level data for PieWithDrilldown |
| PieDrilldownItems | DrilldownSeries[]? | Detail data for PieWithDrilldown |
| PieSortDescending | bool | Sort pie slices largest to smallest (default: true) |
| OnItemClicked | Delegate? | Callback when a chart item is clicked |
| TooltipCallbackHandler | Func<int, string>? | Custom tooltip formatter |

## Resource Preloading

When a page has multiple charts, use a single preload component at the top of the page
to load the Highcharts library once. The other chart components will use the already-loaded
library.

```razor
@* Preload the library without rendering a chart *@
<Highcharts PreloadResources="true" />

@* These charts use the already-loaded library *@
<Highcharts ChartType="Highcharts.ChartTypes.Column" ElementId="chart1" ... />
<Highcharts ChartType="Highcharts.ChartTypes.Pie" ElementId="chart2" ... />
```

## Usage Examples

### Column Chart (Categorical)

```razor
<div id="my-column-chart"></div>
<Highcharts ChartType="Highcharts.ChartTypes.Column"
            ElementId="my-column-chart"
            ChartTitle="Sales by Region"
            ChartSubtitle="Q1 2024"
            yAxisText="Revenue ($)"
            SeriesCategories="_categories"
            SeriesDataArrayItems="_seriesData" />

@code {
    string[] _categories = new[] { "North", "South", "East", "West" };
    
    Highcharts.SeriesDataArray[] _seriesData = new[] {
        new Highcharts.SeriesDataArray {
            name = "2023",
            data = new decimal[] { 100, 200, 150, 180 }
        },
        new Highcharts.SeriesDataArray {
            name = "2024",
            data = new decimal[] { 120, 220, 160, 200 }
        }
    };
}
```

### Pie Chart

```razor
<div id="my-pie-chart"></div>
<Highcharts ChartType="Highcharts.ChartTypes.Pie"
            ElementId="my-pie-chart"
            ChartTitle="Market Share"
            ChartSubtitle="By Product Category"
            SeriesDataItems="_pieData" />

@code {
    Highcharts.SeriesData[] _pieData = new[] {
        new Highcharts.SeriesData { name = "Product A", data = 45.5m },
        new Highcharts.SeriesData { name = "Product B", data = 30.2m },
        new Highcharts.SeriesData { name = "Product C", data = 24.3m }
    };
}
```

### Line Time-Series

```razor
<div id="my-line-chart"></div>
<Highcharts ChartType="Highcharts.ChartTypes.LineTimeSeries"
            ElementId="my-line-chart"
            ChartTitle="Temperature Over Time"
            ChartSubtitle="Drag to zoom"
            yAxisText="°C"
            ShowZoomHint="true"
            SeriesDataXYArrayItems="_temperatureData" />

@code {
    Highcharts.SeriesDataXYArray[] _temperatureData = Array.Empty<Highcharts.SeriesDataXYArray>();

    protected override void OnInitialized()
    {
        var now = DateTimeOffset.UtcNow;
        var indoor = new List<long[]>();
        var outdoor = new List<long[]>();
        
        for (int i = 0; i < 24; i++)
        {
            var timestamp = now.AddHours(-24 + i).ToUnixTimeMilliseconds();
            indoor.Add(new long[] { timestamp, 20 + Random.Shared.Next(-2, 3) });
            outdoor.Add(new long[] { timestamp, 15 + Random.Shared.Next(-5, 6) });
        }

        _temperatureData = new[] {
            new Highcharts.SeriesDataXYArray { name = "Indoor", data = indoor.ToArray() },
            new Highcharts.SeriesDataXYArray { name = "Outdoor", data = outdoor.ToArray() }
        };
    }
}
```

### Column Time-Series

```razor
<div id="my-column-ts-chart"></div>
<Highcharts ChartType="Highcharts.ChartTypes.ColumnTimeSeries"
            ElementId="my-column-ts-chart"
            ChartTitle="Daily Sales"
            ChartSubtitle="Last 30 Days"
            yAxisText="Units Sold"
            ShowZoomHint="true"
            SeriesDataXYArrayItems="_salesData" />

@code {
    Highcharts.SeriesDataXYArray[] _salesData = Array.Empty<Highcharts.SeriesDataXYArray>();

    protected override void OnInitialized()
    {
        var now = DateTimeOffset.UtcNow;
        var sales = new List<long[]>();
        
        for (int i = 0; i < 30; i++)
        {
            var timestamp = now.AddDays(-30 + i).ToUnixTimeMilliseconds();
            sales.Add(new long[] { timestamp, Random.Shared.Next(50, 200) });
        }

        _salesData = new[] {
            new Highcharts.SeriesDataXYArray { name = "Sales", data = sales.ToArray() }
        };
    }
}
```

### Area Totals

```razor
<div id="my-area-chart"></div>
<Highcharts ChartType="Highcharts.ChartTypes.AreaTotals"
            ElementId="my-area-chart"
            ChartTitle="Cumulative Revenue"
            ChartSubtitle="Year to Date"
            yAxisText="Revenue ($)"
            ShowZoomHint="true"
            SeriesDataXYArrayItems="_cumulativeData" />

@code {
    Highcharts.SeriesDataXYArray[] _cumulativeData = Array.Empty<Highcharts.SeriesDataXYArray>();

    protected override void OnInitialized()
    {
        var now = DateTimeOffset.UtcNow;
        var totals = new List<long[]>();
        long runningTotal = 0;
        
        for (int i = 0; i < 12; i++)
        {
            var timestamp = now.AddMonths(-12 + i).ToUnixTimeMilliseconds();
            runningTotal += Random.Shared.Next(10000, 50000);
            totals.Add(new long[] { timestamp, runningTotal });
        }

        _cumulativeData = new[] {
            new Highcharts.SeriesDataXYArray { name = "Total Revenue", data = totals.ToArray() }
        };
    }
}
```

### Pie with Drilldown

The drilldown chart shows a root-level pie chart. When you click a slice, it drills down
to show detail data for that slice. The `DrilldownSeries.id` must match the `SeriesData.name`
exactly for the drilldown to work.

```razor
<div id="my-drilldown-pie"></div>
<Highcharts ChartType="Highcharts.ChartTypes.PieWithDrilldown"
            ElementId="my-drilldown-pie"
            ChartTitle="Sales by Department"
            ChartSubtitle="Click a slice to see product breakdown"
            PieRootItems="_departmentData"
            PieDrilldownItems="_productBreakdown"
            PieSortDescending="true" />

@code {
    Highcharts.SeriesData[] _departmentData = Array.Empty<Highcharts.SeriesData>();
    Highcharts.DrilldownSeries[] _productBreakdown = Array.Empty<Highcharts.DrilldownSeries>();

    protected override void OnInitialized()
    {
        // Root level: Department totals
        _departmentData = new[] {
            new Highcharts.SeriesData { name = "Electronics", data = 45 },
            new Highcharts.SeriesData { name = "Clothing", data = 30 },
            new Highcharts.SeriesData { name = "Home", data = 25 }
        };

        // Drilldown: Product breakdown per department
        // The id must match the root item's name exactly
        _productBreakdown = new[] {
            new Highcharts.DrilldownSeries {
                id = "Electronics",
                data = new[] {
                    new Highcharts.DrilldownPoint { name = "Phones", y = 20 },
                    new Highcharts.DrilldownPoint { name = "Laptops", y = 15 },
                    new Highcharts.DrilldownPoint { name = "Tablets", y = 10 }
                }
            },
            new Highcharts.DrilldownSeries {
                id = "Clothing",
                data = new[] {
                    new Highcharts.DrilldownPoint { name = "Shirts", y = 12 },
                    new Highcharts.DrilldownPoint { name = "Pants", y = 10 },
                    new Highcharts.DrilldownPoint { name = "Shoes", y = 8 }
                }
            },
            new Highcharts.DrilldownSeries {
                id = "Home",
                data = new[] {
                    new Highcharts.DrilldownPoint { name = "Furniture", y = 15 },
                    new Highcharts.DrilldownPoint { name = "Decor", y = 10 }
                }
            }
        };
    }
}
```

## Converting DateTime to Highcharts Timestamp

Highcharts expects timestamps as Unix epoch milliseconds. Use `DateTimeOffset` for conversion:

```csharp
private static long ToEpochMs(DateTime dt) 
    => new DateTimeOffset(dt).ToUnixTimeMilliseconds();
```

## Click Event Handling

```razor
<div id="clickable-pie"></div>
<Highcharts ChartType="Highcharts.ChartTypes.Pie"
            ElementId="clickable-pie"
            ChartTitle="Click a Slice"
            SeriesDataItems="_pieData"
            OnItemClicked="HandlePieClick" />

<div class="alert alert-info">
    Selected: @_selectedItem
</div>

@code {
    Highcharts.SeriesData[] _pieData = new[] {
        new Highcharts.SeriesData { name = "A", data = 30 },
        new Highcharts.SeriesData { name = "B", data = 50 },
        new Highcharts.SeriesData { name = "C", data = 20 }
    };
    
    string _selectedItem = "(none)";

    void HandlePieClick(int index)
    {
        _selectedItem = _pieData[index].name;
        StateHasChanged();
    }
}
```

## Troubleshooting

### Chart Not Rendering

1. Make sure the `ElementId` matches the `id` of an existing `<div>` element
2. Check that data arrays are not empty
3. Check the browser console for JavaScript errors

### Drilldown Not Working

1. Make sure `DrilldownSeries.id` exactly matches `SeriesData.name` (case-sensitive)
2. The drilldown.js module is loaded automatically by the component

### Charts Flickering on Multiple Updates

1. Use preloading: `<Highcharts PreloadResources="true" />`
2. Batch data updates before calling `StateHasChanged()`

### Time-Series X-Axis Shows Wrong Dates

1. Make sure timestamps are in milliseconds, not seconds
2. Use `DateTimeOffset.ToUnixTimeMilliseconds()` for conversion

## Test Page

The HighchartsTest.razor page demonstrates all chart types with live timer-driven data
updates. Navigate to `/HighchartsTest` or `/{TenantCode}/HighchartsTest` to view.

The test page shows:

- Timer controls to adjust update frequency (1-15 seconds) and pause/resume
- Categorical charts (Column, Pie) with rolling window data
- Time-series charts (Line, Column, Area) with datetime x-axis and zoom support
- Pie with drilldown showing click-through detail
- Full history chart that retains all data since page load

See the test page source code for examples of:

- Building time-series data from DateTime values
- Trimming data to maintain a rolling window
- Thread-safe data updates with timer events
- Building drilldown data structures
