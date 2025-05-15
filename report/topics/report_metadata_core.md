# Salesforce Report Metadata: Core Concepts (report_metadata_core.md)

## 1. Overview of Salesforce Report Metadata

Salesforce Report metadata defines the structure, data sources, and presentation of custom reports within the Salesforce platform. This metadata is typically represented in an XML file format, often named with a `.report-meta.xml` suffix (though the Salesforce documentation may refer to the component having a `.report` extension, the actual XML file contains the definition). These files are stored in the `reports` directory within a Salesforce DX project or a metadata package.

Reports allow users to analyze Salesforce data, and their definitions can be retrieved, deployed, and manipulated using the Salesforce Metadata API. Understanding this metadata structure is crucial for developers looking to automate report creation, modify existing reports programmatically, or manage report definitions as part of their development lifecycle.

**Key Characteristics:**

- **Purpose**: Defines custom reports. Standard reports are not supported by this metadata type.
- **File Suffix**: The XML definition is contained in a file typically ending with `.report-meta.xml`. The report component itself has a `.report` suffix.
- **Directory Location**: Report files are stored in the `reports` directory of the package.
- **Root Element**: The XML definition is enclosed within a `<Report>` tag, with the namespace `http://soap.sforce.com/2006/04/metadata`.
- **API Version**: Report metadata components are available in API version 14.0 and later.

A basic report XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Report-specific elements like name, format, reportType, columns, filters, etc. -->
</Report>
```

## 2. Core Report Properties

These are top-level administrative and structural tags that define the fundamental characteristics of a report.

### `<fullName>`

- **Type**: `string`
- **Description**: This field is inherited from the base `Metadata` type. It represents the unique, fully qualified name of the report, including its folder path. The format is typically `FolderAPIName/ReportAPIName`. For reports in unfiled public folders, it might look like `unfiled$public/ReportAPIName`.
- **Example in `package.xml` for retrieval:**
  ```xml
  <types>
      <members>MyReportFolder/MySalesPipelineReport</members>
      <name>Report</name>
  </types>
  ```

### `<name>`

- **Type**: `string`
- **Description**: Required. This is the display name of the report as it appears in the Salesforce UI.
- **Example**:
  ```xml
  <name>Opportunity Pipeline Q3</name>
  ```

### `<description>`

- **Type**: `string`
- **Description**: Optional. A general description for the report, which is displayed with the report name. Maximum 255 characters.
- **Example**:
  ```xml
  <description>Tracks all open opportunities expected to close in the third quarter.</description>
  ```

### `<folderName>`

- **Type**: `string`
- **Description**: The API name of the folder that houses the report. This field is part of how the `fullName` is constructed. Available in API version 35.0 and later.
- **Example**:
  ```xml
  <folderName>SalesReports</folderName>
  ```
  _(Note: `folderName` is specified in the manifest (`package.xml`) as part of the `<members>` tag, or implicitly determined by the directory structure during deployment. Within the `.report-meta.xml` file itself, the `fullName` is the primary identifier for its path.)_

### `<format>`

- **Type**: `ReportFormat` (enumeration of type `string`)
- **Description**: Required. Defines the overall layout and structure of the report.
- **Valid Values**:
  - `Tabular`: A simple listing of data without subtotals. Good for creating lists of records or data to export.
  - `Summary`: Similar to a tabular report but allows grouping of rows of data, viewing subtotals, and creating charts.
  - `Matrix`: Summarizes data in a grid format. Allows grouping by both rows and columns. Useful for comparing related totals.
  - `Joined`: Joins data from different report types, storing each report's data in its own block.
- **Example**:
  ```xml
  <format>Summary</format>
  ```

### `<reportType>`

- **Type**: `string`
- **Description**: Required. Defines the set of objects and fields available for the report. This corresponds to a standard or custom report type in Salesforce (e.g., `Opportunity`, `Account`, or a custom `MyCustomObject__c_RT`).
- **Example**:
  ```xml
  <reportType>Opportunity</reportType>
  ```

### `<division>`

- **Type**: `string`
- **Description**: Optional. If your organization uses divisions to segment data and the "Affected by Divisions" permission is enabled, records in the report must match this division. Available in API version 17.0 and later.
- **Example**:
  ```xml
  <division>NorthAmerica</division>
  ```

### `<scope>`

- **Type**: `string`
- **Description**: Defines the scope of data on which the report runs (e.g., records owned by the user, their team, or all records). Valid values depend on the `reportType`. For example, for Accounts reports, valid values include:
  - `MyAccounts`
  - `MyTeamsAccounts`
  - `AllAccounts`
- **Example (from a full report sample):**
  ```xml
  <scope>organization</scope> <!-- Indicates all records the running user can see -->
  ```

### `<showDetails>`

- **Type**: `boolean`
- **Description**: Determines if the detailed rows are shown in summary or matrix reports.
  - `true` (default): Shows detailed records.
  - `false`: Shows a collapsed view of the report with only headings, subtotals, and totals.
- **Example**:
  ```xml
  <showDetails>false</showDetails>
  ```

## 3. Report Charts (Brief Overview)

Reports can include charts to visually represent the data. The chart definition is contained within the `<chart>` element.

### `<chart>`

- **Description**: Defines the properties of a chart associated with the report. This element is typically used for Summary and Matrix reports.
- **Key Child Elements/Attributes (High-Level)**:

  - `<chartType>`: (Required) Specifies the type of chart (e.g., `Bar`, `Line`, `Pie`, `VerticalColumn`). This is an enumeration of type `ChartType`.
  - `<location>`: (Required) Specifies where the chart is displayed in the report (e.g., `CHART_TOP`, `CHART_BOTTOM`). This is an enumeration of type `ChartPosition`.
  - `<size>`: (Required) Specifies the size of the chart (e.g., `Small`, `Medium`, `Large`). This is an enumeration of type `ReportChartSize`.
  - `<title>`: An optional title for the chart (max 255 characters).
  - Styling attributes: Such as `<backgroundColor1>`, `<backgroundColor2>`, `<backgroundFadeDir>`, `<textColor>`, `<textSize>`, `<titleColor>`, `<titleSize>`.
  - Data summary attributes: Such as `<chartSummaries>` (which define what data is plotted), `<groupingColumn>` (for the X-axis or categories).

- **Example Snippet**:
  ```xml
  <chart>
      <chartType>VerticalColumn</chartType>
      <location>CHART_TOP</location>
      <size>Medium</size>
      <title>Sales by Quarter</title>
      <groupingColumn>FISCAL_QUARTER</groupingColumn>
      <chartSummaries>
          <axisBinding>y</axisBinding>
          <aggregate>Sum</aggregate>
          <column>AMOUNT</column>
      </chartSummaries>
      <!-- Other chart settings -->
  </chart>
  ```
  _(A more detailed explanation of chart metadata will be covered in subsequent documents if extensive chart details are available in the source.)_

---

This document provides an overview of the core metadata components for Salesforce Reports. For more specific details on data shaping (columns, filters) and summarization (groupings, aggregates), refer to the subsequent documentation files.
