# Salesforce Report Metadata: Summarization and Grouping (report_summarization_grouping.md)

## 1. Introduction to Summarization and Grouping

In Salesforce reports, summarization and grouping are key features for analyzing data. Grouping allows you to organize report data by specific field values, while summarization provides aggregate calculations (like sum, average, count) for these groups and for the report as a whole. This is particularly relevant for Summary and Matrix report formats.

This document details the metadata elements used to define row and column groupings, as well as aggregate summaries and custom summary formulas.

## 2. Defining Report Groupings

Reports can be grouped by rows (`<groupingsDown>`) and, in the case of Matrix reports, also by columns (`<groupingsAcross>`). These groupings determine how data is segmented and subtotaled.

**Structure (Type `ReportGrouping`):**

```xml
<groupingsDown> <!-- Or <groupingsAcross> for Matrix reports -->
    <dateGranularity>Month</dateGranularity> <!-- Optional: For grouping by date fields -->
    <field>Opportunity.StageName</field>    <!-- Required: API name of the field to group by -->
    <sortOrder>Asc</sortOrder>              <!-- Required: Asc or Desc -->
    <!-- Optional elements for sorting by aggregate values: -->
    <!-- <aggregateType>Sum</aggregateType> -->
    <!-- <sortByName>AMOUNT</sortByName> --> <!-- API name of the column, aggregate, or CSF -->
    <!-- <sortType>Aggregate</sortType> -->   <!-- Column, Aggregate, or CustomSummaryFormula -->
</groupingsDown>
```

**Key Sub-Elements of `<groupingsDown>` and `<groupingsAcross>` (type `ReportGrouping`):**

- **`<field>`** (`string`, Required):
  - The API name of the field by which you want to group and subtotal data.
  - Example: `Opportunity.StageName`, `Account.Industry`.
- **`<sortOrder>`** (`SortOrder`, Required):
  - Specifies whether to sort the grouped data in ascending (`Asc`) or descending (`Desc`) alphabetical or numerical order.
- **`<dateGranularity>`** (`UserDateGranularity`, Optional):
  - When grouping by a date field, this specifies the time period by which to group (e.g., `Day`, `Week`, `Month`, `Quarter`, `Year`, `FiscalQuarter`, `FiscalYear`).
  - Valid values are defined by the `UserDateGranularity` enumeration.
- **`<aggregateType>`** (`ReportAggrType`, Optional):
  - When `sortType` is `Aggregate` or `CustomSummaryFormula`, this specifies the type of aggregate value to sort by.
  - Valid values: `Sum`, `Average`, `Maximum`, `Minimum`, `RowCount`, `Unique`, `Median`, `Noop`.
- **`<sortByName>`** (`string`, Optional):
  - The API name of the column, aggregate, or custom summary field used to order the grouping when `sortType` is not `Column`.
- **`<sortType>`** (`ReportSortType`, Optional):
  - Indicates if the grouping is sorted by a column (default), an aggregate value, or a custom summary field.
  - Valid values: `Column`, `Aggregate`, `CustomSummaryFormula`.

**Usage:**

- **Summary Reports:** Use `<groupingsDown>`. You can have up to 3 levels of row groupings.
- **Matrix Reports:** Use `<groupingsDown>` (for row headings) and `<groupingsAcross>` (for column headings). You can have up to 2 levels for each.

**Example of Multiple Row Groupings:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- ... other report elements ... -->
    <format>Summary</format>
    <groupingsDown>
        <dateGranularity>FiscalQuarter</dateGranularity>
        <field>Opportunity.CloseDate</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <groupingsDown>
        <field>Opportunity.Type</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <!-- ... columns, filters ... -->
</Report>
```

**Example of Matrix Groupings:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- ... other report elements ... -->
    <format>Matrix</format>
    <groupingsAcross>
        <dateGranularity>Month</dateGranularity>
        <field>Opportunity.CloseDate</field>
        <sortOrder>Asc</sortOrder>
    </groupingsAcross>
    <groupingsDown>
        <field>Opportunity.Owner.Name</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <!-- ... columns (typically summarized fields for matrix cells), filters ... -->
</Report>
```

## 3. Defining Aggregates and Custom Summary Formulas (`<aggregates>`)

The `<aggregates>` element (of type `ReportAggregate`) defines custom summary formulas (CSFs) for summary, matrix, and joined reports. These formulas allow you to perform calculations based on summarized data.

**Structure:**

```xml
<aggregates>
    <calculatedFormula>AMOUNT:SUM * 0.10</calculatedFormula> <!-- Required: The formula logic -->
    <datatype>currency</datatype>                          <!-- Required: currency, number, percent, text -->
    <developerName>CommissionCSF</developerName>             <!-- Required: Unique internal name -->
    <isActive>true</isActive>                              <!-- Required: true or false -->
    <masterLabel>Commission Amount</masterLabel>              <!-- Required: Display label -->
    <scale>2</scale>                                      <!-- Required: Decimal places (0-18) -->
    <!-- Optional for display context in matrix/summary reports (API v15.0+): -->
    <!-- <acrossGroupingContext>GRAND_SUMMARY</acrossGroupingContext> -->
    <!-- <downGroupingContext>GRAND_SUMMARY</downGroupingContext> -->
    <!-- Optional for joined reports (API v25.0+): -->
    <!-- <isCrossBlock>false</isCrossBlock> -->
    <!-- <reportType>Opportunity</reportType> --> <!-- For standard CSFs in joined reports -->
    <description>Calculates 10% commission on summed amount.</description> <!-- Optional -->
</aggregates>
```

**Key Sub-Elements of `<aggregates>` (type `ReportAggregate`):**

- **`<calculatedFormula>`** (`string`, Required):
  - The custom summary formula logic.
  - Examples: `AMOUNT:SUM + OPP_QUANTITY:SUM`, `RowCount / PARENTGROUPVAL(RowCount, GRAND_SUMMARY)`.
  - For cross-block custom summary formulas in joined reports, it references block ID and aggregate: `B1#AMOUNT:SUM + B2#RowCount`.
- **`<datatype>`** (`ReportAggregateDatatype`, Required):
  - Specifies the data type for formatting and display of the formula results.
  - Valid values: `currency`, `number`, `percent`.
- **`<developerName>`** (`string`, Required):
  - The unique internal development name for the custom summary formula (e.g., `FORMULA1`). This name is used to reference the CSF from other report components, like conditional highlighting or charts.
- **`<isActive>`** (`boolean`, Required):
  - `true` displays the formula result in the report. `false` does not.
- **`<masterLabel>`** (`string`, Required):
  - The display label for the custom summary formula in the report.
- **`<scale>`** (`int`, Required):
  - The number of decimal places for the formula result (valid values 0 through 18).
- **`<description>`** (`string`, Optional):
  - A description for the custom summary formula (max 255 characters).
- **`<acrossGroupingContext>`** (`string`, Optional, API v15.0+):
  - For matrix reports, defines the column grouping level at which this CSF should be displayed. Can be a specific grouping field API name or `GRAND_SUMMARY`.
- **`<downGroupingContext>`** (`string`, Optional, API v15.0+):
  - For summary or matrix reports, defines the row grouping level at which this CSF should be displayed. Can be a specific grouping field API name or `GRAND_SUMMARY`.
- **`<isCrossBlock>`** (`boolean`, Optional, API v25.0+):
  - Determines if the CSF is a cross-block formula (used in joined reports). `true` for cross-block, `false` for standard.
- **`<reportType>`** (`string`, Optional, for Joined Reports):
  - Required for standard (non-cross-block) CSFs in joined reports. Specifies the `reportType` of the block to which this aggregate can be added.

**Example of a Custom Summary Formula:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- ... reportType, format, groupingsDown ... -->
    <columns>
        <field>OPPORTUNITY.NAME</field>
    </columns>
    <columns>
        <aggregateTypes>Sum</aggregateTypes>
        <field>AMOUNT</field>
    </columns>
    <columns>
        <aggregateTypes>Sum</aggregateTypes>
        <field>Opportunity.ExpectedRevenue</field>
    </columns>

    <aggregates>
        <calculatedFormula>AMOUNT:SUM - ExpectedRevenue:SUM</calculatedFormula>
        <datatype>currency</datatype>
        <developerName>DiffAmountExpected</developerName>
        <downGroupingContext>GRAND_SUMMARY</downGroupingContext> <!-- Display at grand total level -->
        <isActive>true</isActive>
        <masterLabel>Difference (Amount - Expected)</masterLabel>
        <scale>2</scale>
    </aggregates>
    <!-- ... filters ... -->
</Report>
```

## 4. Grand Totals and Subtotals

Grand totals and subtotals are implicitly controlled by the report format and the presence of groupings and summarized fields.

- **`<showGrandTotal>`** (`boolean`, Optional):
  - `true` (default) displays the calculated grand total for the full report.
  - Part of the main report definition.
- **`<showSubTotals>`** (`boolean`, Optional):
  - `true` (default) displays the calculated subtotals for sections (groups) of the report.
  - Part of the main report definition.

**Example:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- ... other elements ... -->
    <showGrandTotal>true</showGrandTotal>
    <showSubTotals>true</showSubTotals>
    <!-- ... -->
</Report>
```

This document covers how to define groupings for rows and columns in Salesforce reports and how to create aggregate summaries and custom summary formulas to perform calculations on your grouped data.

## 5. Joined Report Structure Details

Joined reports allow combining data from multiple different report types into a single view, presenting each report type's data in its own "block". The overall report format is set using `<format>MultiBlock</format>`.

**Key Structural Elements:**

- **`<block>`**: Represents a single report block within the joined report. Each block contains metadata similar to a standalone Summary report.
- **`<blockInfo>`**: Contains metadata linking blocks together and referencing formulas. Present within each `<block>` and also at the root level of the report definition.
- **`<joinTable>`**: Specifies the common object or entity used to link the data across different blocks (often represented as `a`, potentially signifying Account if it's the common link).

**Structure of a `<block>` Element:**

```xml
<block>
    <blockInfo>
        <aggregateReferences>
            <aggregate>FORMULA2</aggregate> <!-- References a STANDARD CSF developerName -->
        </aggregateReferences>
        <blockId>B1</blockId> <!-- Required, Unique ID for this block (B1-B5) -->
        <joinTable>a</joinTable> <!-- Required, Link entity -->
    </blockInfo>
    <columns>...</columns> <!-- Columns specific to this block -->
    <filter>...</filter> <!-- Filters specific to this block -->
    <format>Summary</format> <!-- Usually Summary for individual blocks -->
    <name>Block 1 Name</name> <!-- Name for this block -->
    <params> <!-- Optional parameters specific to the block's report type -->
      <name>parameter_api_name</name>
      <value>parameter_value</value>
    </params>
    <reportType>Opportunity</reportType> <!-- Required: Report type for this block -->
    <scope>organization</scope> <!-- Scope for this block -->
    <timeFrameFilter>...</timeFrameFilter> <!-- Time frame filter for this block -->
</block>
```

**Key Sub-Elements of `<blockInfo>`:**

- **`<blockId>`** (`string`, Required within `<block>`): A unique identifier for the block (e.g., `B1`, `B2`). This ID is used in cross-block custom summary formulas (e.g., `B1#AMOUNT:SUM`). The root-level `<blockInfo>` typically has `<blockId xsi:nil="true"/>`.
- **`<joinTable>`** (`string`, Required): Specifies the entity used to join blocks. The source examples use `a`.
- **`<aggregateReferences>`** (`ReportAggregateReference[]`, Optional): Used within a block's `<blockInfo>` to link _standard_ (non-cross-block) Custom Summary Formulas (`<aggregates>`) to this specific block.
  - **`<aggregate>`** (`string`, Required): The `developerName` of the standard `ReportAggregate` to be displayed or used within this block.

**`<params>` (`ReportParam`)**

- Used within a `<block>` to specify settings specific to that block's `reportType`.
- Allows filtering the block to useful subsets (e.g., show only 'Open' Activities).
- **`<name>`** (`string`, Required): The specific setting name (depends on `reportType`).
- **`<value>`** (`string`, Required): The value for the setting.

**Cross-Block Formulas in Joined Reports:**

- As covered previously, Custom Summary Formulas (`<aggregates>`) with `<isCrossBlock>true</isCrossBlock>` are defined globally for the joined report.
- Their `<calculatedFormula>` uses the `blockId#aggregateName` syntax (e.g., `B1#AMOUNT:SUM + B2#RowCount`) to reference summary values from specific blocks.
