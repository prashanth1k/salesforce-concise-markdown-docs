## 1. Introduction

This document delves into the detailed configuration options for charts within Salesforce reports and touches upon other advanced metadata features like conditional highlighting.

## 2. Advanced Chart Configuration (`<chart>`)

While the basics are covered elsewhere, the `<chart>` element offers numerous options for fine-tuning chart appearance and behavior.

**Structure Revisited:**

```xml
<chart>
    <!-- Core Type, Location, Size -->
    <chartType>VerticalColumnGroupedLine</chartType>
    <location>CHART_BOTTOM</location>
    <size>Large</size>

    <!-- Background & Colors -->
    <backgroundColor1>#FFFFFF</backgroundColor1>
    <backgroundColor2>#DDDDDD</backgroundColor2>
    <backgroundFadeDir>LeftToRight</backgroundFadeDir>

    <!-- Data Summaries & Axes -->
    <chartSummaries>
        <aggregate>Sum</aggregate>
        <axisBinding>y</axisBinding> <!-- Primary Y-axis -->
        <column>AMOUNT</column>
    </chartSummaries>
    <chartSummaries>
        <aggregate>Average</aggregate>
        <axisBinding>y2</axisBinding> <!-- Secondary Y-axis (for combo charts) -->
        <column>PROBABILITY</column>
    </chartSummaries>
    <groupingColumn>STAGE_NAME</groupingColumn> <!-- X-axis / Category -->
    <secondaryGroupingColumn>CLOSE_DATE</secondaryGroupingColumn> <!-- Second grouping for grouped chart types -->

    <!-- Axis Configuration -->
    <summaryAxisRange>Manual</summaryAxisRange> <!-- Auto or Manual -->
    <summaryAxisManualRangeStart>0</summaryAxisManualRangeStart>
    <summaryAxisManualRangeEnd>100000</summaryAxisManualRangeEnd>

    <!-- Legend & Labels -->
    <legendPosition>Bottom</legendPosition> <!-- Bottom, OnChart, Right -->
    <enableHoverLabels>true</enableHoverLabels>
    <showAxisLabels>true</showAxisLabels>

    <!-- Display Toggles -->
    <showPercentage>false</showPercentage> <!-- For Pie, Donut, Funnel -->
    <showTotal>false</showTotal> <!-- For Donut, Gauge -->
    <showValues>true</showValues> <!-- Show data values on chart elements -->
    <expandOthers>false</expandOthers> <!-- Combine small groups into 'Others' (Pie/Donut/Funnel)? -->

    <!-- Text Styling -->
    <textColor>#333333</textColor>
    <textSize>10</textSize>
    <title>Opportunity Analysis by Stage and Quarter</title>
    <titleColor>#000000</titleColor>
    <titleSize>14</titleSize>
</chart>
```

**Detailed Chart Configuration Elements:**

- **`<chartSummaries>` (`ChartSummary[]`)**: Defines what data is plotted.
  - **`<aggregate>`** (`ReportSummaryType`): The aggregation method (e.g., `Sum`, `Average`, `Max`, `Min`). Not needed for `RowCount` or custom summary formulas referenced by `column`.
  - **`<axisBinding>`** (`ChartAxis`): Specifies the axis (`x`, `y`, `y2`). `y2` is for the secondary Y-axis in combination charts. `x` is used for the X-axis summary value in scatter charts.
  - **`<column>`** (`string`, Required): The API name of the summarized field or `RowCount`. For vertical/horizontal bar combo charts, you can specify up to four values.
- **`<groupingColumn>`** (`string`): Specifies the field for the primary grouping (typically X-axis categories or horizontal bar groups).
- **`<secondaryGroupingColumn>`** (`string`): For grouped chart types, specifies the field for the secondary grouping (e.g., segments within a bar).
- **`<chartType>`** (`ChartType`): Defines the chart visual type (e.g., `HorizontalBar`, `VerticalColumnStacked`, `LineGrouped`, `Pie`, `Donut`, `Funnel`, `Scatter`). See `ChartType` enumeration for full list.
- **`<size>`** (`ReportChartSize`): Defines chart dimensions. Valid values: `Tiny`, `Small`, `Medium`, `Large`, `Huge`.
- **`<location>`** (`ChartPosition`): Where the chart appears. Valid values: `CHART_TOP`, `CHART_BOTTOM`.
- **`<legendPosition>`** (`ChartLegendPosition`): Legend placement. Valid values: `Bottom`, `OnChart`, `Right`.
- **`<summaryAxisRange>`** (`ChartRangeType`): Axis scale. Valid values: `Auto`, `Manual`.
- **`<summaryAxisManualRangeStart>` / `<summaryAxisManualRangeEnd>`** (`double`): Defines the start/end values when `summaryAxisRange` is `Manual`.
- **`<backgroundFadeDir>`** (`ChartBackgroundDirection`): Gradient direction. Valid values: `Diagonal`, `LeftToRight`, `TopToBottom`. Use with `<backgroundColor1>` and `<backgroundColor2>`.
- **`<enableHoverLabels>`** (`boolean`, API v17.0+): Show details on hover.
- **`<expandOthers>`** (`boolean`, API v17.0+): Combine small groups (<=3%) into an 'Others' segment (Pie/Donut/Funnel only). `false` combines, `true` shows all individually.
- **`<showAxisLabels>`** (`boolean`): Display names for axes (Bar/Line).
- **`<showPercentage>`** (`boolean`): Display percentages (Pie/Donut/Funnel/Gauge).
- **`<showTotal>`** (`boolean`): Display total in the center (Donut/Gauge).
- **`<showValues>`** (`boolean`): Display data values on chart elements.
- **Text/Color Styling**: `<textColor>`, `<textSize>`, `<titleColor>`, `<titleSize>` control the appearance of chart text and titles using standard hex color codes and point sizes (e.g., 8, 9, 10, 12, 14, 18, 24, 36 - max displayed is 18pt).

## 3. Conditional Highlighting (Lightning - `<formattingRules>`)

_(Note: This section details Lightning conditional highlighting. Classic uses `<colorRanges>`)_

Conditional highlighting applies background colors to summarized report data based on defined thresholds. You can specify up to 5 formatting rules per report.

**Structure (within the main `<Report>` element):**

```xml
<formattingRules>
    <aggregate>Sum</aggregate> <!-- Required: Summary type the rule applies to -->
    <columnName>AMOUNT</columnName> <!-- Required: The summarized column -->
    <!-- Up to 3 <values> elements defining color ranges -->
    <values>
      <backgroundColor>#FF9897</backgroundColor> <!-- Optional: Hex color -->
      <rangeUpperBound>10000.0</rangeUpperBound> <!-- Optional: Upper bound for this color -->
    </values>
    <values>
      <backgroundColor>#FFFF93</backgroundColor>
      <rangeUpperBound>50000.0</rangeUpperBound>
    </values>
    <values>
      <backgroundColor>#93FF93</backgroundColor>
      <!-- No rangeUpperBound means this applies from 50000.0 to infinity -->
    </values>
</formattingRules>
```

**Key Elements (`ReportFormattingRule`):**

- **`<aggregate>`** (`ReportFormattingSummaryType`, Required): Defines how the target field is summarized for the rule. Valid values: `Sum`, `Average`, `Maximum`, `Minimum`, `Unique`.
- **`<columnName>`** (`string`, Required): The API name of the summarized field the rule applies to.
- **`<values>`** (`ReportFormattingRuleValue[]`, Required, 1-3): Defines a specific color band.
  - **`<backgroundColor>`** (`string`): The hex color code (e.g., `#B50E03`). If omitted for a range, the background is transparent. At least one color is required across all `<values>`.
  - **`<rangeUpperBound>`** (`double`): Delineates the upper boundary for this color range. If omitted, the range extends to infinity. The ranges are applied sequentially based on the upper bounds.

## 4. Conditional Highlighting (Classic - `<colorRanges>`)

This element defines conditional highlighting specifically for Salesforce Classic summary data.

**Structure (within the main `<Report>` element):**

```xml
<colorRanges>
    <aggregate>Sum</aggregate> <!-- Required: e.g., Sum, Average, Min, Max -->
    <columnName>AMOUNT</columnName> <!-- Required: The summarized column -->
    <highBreakpoint>50000.0</highBreakpoint> <!-- Required: Threshold between mid and high color -->
    <highColor>#00FF00</highColor> <!-- Required: Hex color for high range -->
    <lowBreakpoint>10000.0</lowBreakpoint> <!-- Required: Threshold between low and mid color -->
    <lowColor>#FF0000</lowColor> <!-- Required: Hex color for low range -->
    <midColor>#FFFF00</midColor> <!-- Required: Hex color for mid range -->
</colorRanges>
```

**Key Elements (`ReportColorRange`):**

- **`<aggregate>`** (`ReportSummaryType`, Required): How the field is summarized.
- **`<columnName>`** (`string`, Required): The summarized field.
- **`<highBreakpoint>`** (`double`, Required): Separates mid from high color range.
- **`<highColor>`** (`string`, Required): HTML color for values >= `highBreakpoint`.
- **`<lowBreakpoint>`** (`double`, Required): Separates low from mid color range.
- **`<lowColor>`** (`string`, Required): HTML color for values < `lowBreakpoint`.
- **`<midColor>`** (`string`, Required): HTML color for values >= `lowBreakpoint` and < `highBreakpoint`.
