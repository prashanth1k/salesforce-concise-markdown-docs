# Salesforce Report Metadata: Data Shaping (report_data_shaping.md)

## 1. Introduction to Data Shaping in Reports

Data shaping in Salesforce Report metadata refers to the definition of which fields are displayed (columns), how the data is filtered to include only relevant records, and how related data is included or excluded (cross-filters). These components are crucial for tailoring the report output to specific analytical needs.

This document covers the metadata elements used to define columns and various types of filters.

## 2. Defining Report Columns (`<columns>`)

The `<columns>` element is used to specify which fields from the report type are displayed as columns in the report. You can list multiple `<columns>` elements, and they will appear in the report in the same order as they are defined in the metadata.

Each `<columns>` element typically contains a `<field>` sub-element. For summary or matrix reports, it can also include `<aggregateTypes>` to define how the data in that column should be summarized.

**Structure:**

```xml
<columns>
    <aggregateTypes>Sum</aggregateTypes> <!-- Optional: Defines how the field is summarized, e.g., Sum, Average, Min, Max -->
    <field>Opportunity.Amount</field>   <!-- Required: API name of the field to display -->
    <!-- For historical trend reports, specific boolean flags might be available: -->
    <!-- <reverseColors>false</reverseColors> -->
    <!-- <showChanges>false</showChanges> -->
</columns>
```

**Key Sub-Elements of `<columns>` (which is of type `ReportColumn`):**

- **`<field>`** (`string`, Required):
  - The API name of the field to be displayed. For standard objects, this is `ObjectName.FieldName` (e.g., `Opportunity.Amount`). For custom objects/fields, it would be `CustomObjectName__c.CustomFieldName__c`.
  - For fields from related objects (e.g., Account Name on an Opportunity report), it follows the relationship path: `Opportunity.Account.Name`.
  - Bucket fields are also referenced here using their developer name, e.g., `BucketField_Region`.
- **`<aggregateTypes>`** (`ReportSummaryType[]`, Optional):
  - A list that defines if and how this report field is summarized when the report format supports it (e.g., Summary, Matrix).
  - Valid `ReportSummaryType` values include: `Sum`, `Average`, `Maximum`, `Minimum`, `Unique`, `Median`, `Noop` (no operation/summary), `None` (field isn't summarized).
- **`<reverseColors>`** (`boolean`, Optional, API v29.0+):
  - In historical trend reports, displays greater Date values as green and greater Amount values as red, reversing the default colors.
- **`<showChanges>`** (`boolean`, Optional, API v29.0+):
  - In historical trend reports, adds a column displaying the difference between current and historical Date and Amount values.

**Example:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- ... other report elements ... -->
    <columns>
        <field>ACCOUNT.NAME</field>
    </columns>
    <columns>
        <field>OPPORTUNITY.NAME</field>
    </columns>
    <columns>
        <aggregateTypes>Sum</aggregateTypes>
        <field>AMOUNT</field> <!-- Field name relative to the primary object of the report type -->
    </columns>
    <columns>
        <aggregateTypes>Average</aggregateTypes>
        <field>AGE</field>
    </columns>
    <columns>
        <field>BucketField_SalesStageGroup</field> <!-- A custom bucket field -->
    </columns>
</Report>
```

## 3. Defining Report Filters (`<filter>`)

The `<filter>` element (of type `ReportFilter`) allows you to limit the report results to records that meet specific criteria. This is essential for focusing the report on the desired data subset.

**Structure:**

```xml
<filter>
    <criteriaItems>
        <column>Opportunity.StageName</column>
        <columnToColumn>false</columnToColumn> <!-- Optional, default false -->
        <isUnlocked>false</isUnlocked>         <!-- Optional, default false, API v38.0+ -->
        <operator>equals</operator>
        <value>Closed Won</value>
        <!-- <snapshot>N_DAYS_AGO:30</snapshot> --> <!-- Optional, for historical trend filters -->
    </criteriaItems>
    <criteriaItems>
        <column>Opportunity.Amount</column>
        <operator>greaterThan</operator>
        <value>50000</value>
    </criteriaItems>
    <booleanFilter>1 AND 2</booleanFilter> <!-- Optional: Defines complex filter logic -->
    <language>en_US</language>             <!-- Optional: For picklist filters with 'contains' or 'startsWith' -->
</filter>
```

**Key Sub-Elements of `<filter>` (type `ReportFilter`):**

- **`<criteriaItems>`** (`ReportFilterItem[]`, Required):
  - One or more criteria for filtering. Each `criteriaItem` defines a single filter condition.
  - **Sub-Elements of `<criteriaItems>` (type `ReportFilterItem`):**
    - **`<column>`** (`string`, Required): The API name of the field to filter on.
    - **`<columnToColumn>`** (`boolean`, Optional):
      - Indicates whether the filter is a column-to-column (field-to-field) comparison.
      - Available in API v29.0+ for historical trending reports, and API v48.0+ for general reports. Default is `false`.
    - **`<isUnlocked>`** (`boolean`, Optional, API v38.0+):
      - Indicates whether the report filter is unlocked (`true`) or locked (`false`). Unlocked filters can be edited on the report run page in Lightning Experience. If unspecified, the default value is `false`.
    - **`<operator>`** (`FilterOperation`, Required): An enumeration of type string that defines the comparison operator. Valid values include:
      - `equals`
      - `notEqual`
      - `lessThan`
      - `greaterThan`
      - `lessOrEqual`
      - `greaterOrEqual`
      - `contains`
      - `notContain`
      - `startsWith`
      - `includes` (for multi-select picklists)
      - `excludes` (for multi-select picklists)
      - `within` (for DISTANCE criteria only, e.g., location-based filters)
    - **`<value>`** (`string`, Optional): The value to compare the field against.
      - The Metadata API filter condition values don't always match the values entered in the report wizard. For example, dates are always converted to US date format (`yyyy-MM-dd`), and values entered in a non-US English language can be converted to a standard US English equivalent.
    - **`<snapshot>`** (`string`, Optional, API v29.0+):
      - For historical trending reports, represents the date value to apply for a historical filter.
      - Can be relative (e.g., `N_DAYS_AGO:7`, `N_WEEKS_AGO:2`) or absolute (e.g., `yyyy-MM-dd`).
- **`<booleanFilter>`** (`string`, Optional):
  - Specifies custom filter logic using AND, OR, and parentheses with numbered criteria (e.g., `1 AND (2 OR 3)`). The numbers correspond to the order of the `<criteriaItems>` elements.
- **`<language>`** (`Language`, Optional):
  - The language used when a report filters against a picklist value using the operators `contains` or `startsWith`. For a list of valid language values, refer to the Salesforce `Language` documentation.

**Example of Filter with Boolean Logic:**

```xml
<filter>
    <criteriaItems> <!-- Criterion 1 -->
        <column>INDUSTRY</column>
        <operator>equals</operator>
        <value>Technology</value>
    </criteriaItems>
    <criteriaItems> <!-- Criterion 2 -->
        <column>ANNUAL_REVENUE</column>
        <operator>greaterThan</operator>
        <value>1000000</value>
    </criteriaItems>
    <criteriaItems> <!-- Criterion 3 -->
        <column>TYPE</column>
        <operator>equals</operator>
        <value>Customer - Direct</value>
    </criteriaItems>
    <booleanFilter>(1 AND 2) OR 3</booleanFilter>
</filter>
```

## 4. Defining Cross-Filters (`<crossFilters>`)

Cross-filters allow you to filter a report based on the related objects and their fields, even if those fields are not directly included in the report. For example, you can find "Accounts WITH Opportunities" or "Contacts WITHOUT Cases". This feature is available in API version 63.0 and later.

**Structure:**

```xml
<crossFilters>
    <criteriaItems> <!-- Optional: Sub-filters on the related object -->
        <column>Status</column>      <!-- Field from the relatedTable -->
        <operator>notequal</operator>
        <value>Closed</value>
    </criteriaItems>
    <operation>with</operation> <!-- 'with' or 'without' -->
    <primaryTableColumn>ACCOUNT_ID</primaryTableColumn> <!-- Join field from the primary report object -->
    <relatedTable>Case</relatedTable> <!-- API name of the related object -->
    <relatedTableJoinColumn>AccountId</relatedTableJoinColumn> <!-- Join field from the relatedTable -->
</crossFilters>
```

**Key Sub-Elements of `<crossFilters>` (type `ReportCrossFilter`):**

- **`<operation>`** (`ObjectFilterOperator`, Required):
  - Indicates whether to include or exclude records from the primary object based on the related object criteria.
  - Valid values: `with`, `without`.
- **`<primaryTableColumn>`** (`string`, Required):
  - The API name of the field from the primary object used for the cross-filter join (typically an ID field).
- **`<relatedTable>`** (`string`, Required):
  - The API name of the child object used for the cross-filter.
- **`<relatedTableJoinColumn>`** (`string`, Required):
  - The API name of the field from the `relatedTable` (child object) that is used to join to the `primaryTableColumn`.
- **`<criteriaItems>`** (`ReportFilterItem[]`, Optional):
  - Represents sub-filters applied to the `relatedTable`. These define conditions that records in the related object must meet.
  - There can be up to five sub-filters.
  - The structure of these `criteriaItems` is the same as those used in standard report filters (see `ReportFilterItem` above), referencing fields from the `relatedTable`.

**Example:**
Find Accounts that have at least one open Case.

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <reportType>Account</reportType>
    <!-- ... other report elements ... -->
    <crossFilters>
        <criteriaItems>
            <column>IsClosed</column> <!-- Case.IsClosed -->
            <operator>equals</operator>
            <value>false</value>
        </criteriaItems>
        <operation>with</operation>
        <primaryTableColumn>ACCOUNT.ID</primaryTableColumn>
        <relatedTable>Case</relatedTable>
        <relatedTableJoinColumn>AccountId</relatedTableJoinColumn>
    </crossFilters>
    <!-- ... columns, etc. ... -->
</Report>
```

## 5. Defining Time Frame Filters (`<timeFrameFilter>`)

The `<timeFrameFilter>` element (of type `ReportTimeFrameFilter`) limits report results to records within a specified time frame. This is often used for filtering based on date fields like Close Date, Created Date, etc.

**Structure:**

```xml
<timeFrameFilter>
    <dateColumn>Opportunity.CloseDate</dateColumn> <!-- Required: The date field to filter on -->
    <endDate>2023-12-31</endDate>           <!-- Optional: End date for INTERVAL_CUSTOM -->
    <interval>INTERVAL_CUSTOM</interval>      <!-- Required: Defines the period -->
    <startDate>2023-01-01</startDate>         <!-- Optional: Start date for INTERVAL_CUSTOM -->
</timeFrameFilter>
```

**Key Sub-Elements of `<timeFrameFilter>` (type `ReportTimeFrameFilter`):**

- **`<dateColumn>`** (`string`, Required):
  - The API name of the date field on which to filter data.
  - Example: `Opportunity.CloseDate`, `Case.CreatedDate`.
- **`<interval>`** (`UserDateInterval`, Required):
  - An enumeration of type string that specifies the predefined or custom time period.
  - See the comprehensive list of valid values below.
- **`<startDate>`** (`date`, Optional):
  - When `interval` is `INTERVAL_CUSTOM`, this specifies the start date of the custom time period (format: `YYYY-MM-DD`).
- **`<endDate>`** (`date`, Optional):
  - When `interval` is `INTERVAL_CUSTOM`, this specifies the end date of the custom time period (format: `YYYY-MM-DD`).

### Valid Values for `<interval>` (`UserDateInterval`)

The `<interval>` element accepts a wide range of predefined values. Based on the `UserDateInterval` enumeration (from OCR pages 22-24 of the source material):

**Custom Interval:**

- `INTERVAL_CUSTOM`: A custom time period. Use `startDate` and `endDate` fields to specify the time period's start date and end date.

**Fiscal Period Intervals:**

- `INTERVAL_CURRENT` (Current fiscal quarter)
- `INTERVAL_CURNEXT1` (Current and next fiscal quarters)
- `INTERVAL_CURPREV1` (Current and previous fiscal quarters)
- `INTERVAL_NEXT1` (Next fiscal quarter)
- `INTERVAL_PREV1` (Previous fiscal quarter)
- `INTERVAL_CURNEXT3` (Current and next three fiscal quarters)
- `INTERVAL_CURFY` (Current fiscal year)
- `INTERVAL_PREVFY` (Previous fiscal year)
- `INTERVAL_PREV2FY` (Previous two fiscal years)
- `INTERVAL_AGO2FY` (Two fiscal years ago)
- `INTERVAL_NEXTFY` (Next fiscal year)
- `INTERVAL_PREVCURFY` (Current and previous fiscal years)
- `INTERVAL_PREVCUR2FY` (Current and previous two fiscal years)
- `INTERVAL_CURNEXTFY` (Current and next fiscal year)
- `LAST_FISCALWEEK` (When custom fiscal years are enabled: Last fiscal week)
- `THIS_FISCALWEEK` (When custom fiscal years are enabled: This fiscal week)
- `NEXT_FISCALWEEK` (When custom fiscal years are enabled: Next fiscal week)
- `LAST_FISCALPERIOD` (When custom fiscal years are enabled: Last fiscal period)
- `THIS_FISCALPERIOD` (When custom fiscal years are enabled: This fiscal period)
- `NEXT_FISCALPERIOD` (When custom fiscal years are enabled: Next fiscal period)
- `LASTTHIS_FISCALPERIOD` (When custom fiscal years are enabled: This fiscal period and last fiscal period)
- `THISNEXT_FISCALPERIOD` (When custom fiscal years are enabled: This fiscal period and next fiscal period)

**Calendar Period Intervals:**

- `INTERVAL_YESTERDAY`
- `INTERVAL_TODAY`
- `INTERVAL_TOMORROW`
- `INTERVAL_LASTWEEK` (Last calendar week)
- `INTERVAL_THISWEEK` (This calendar week)
- `INTERVAL_NEXTWEEK` (Next calendar week)
- `INTERVAL_LASTMONTH` (Last calendar month)
- `INTERVAL_THISMONTH` (This calendar month)
- `INTERVAL_NEXTMONTH` (Next calendar month)
- `INTERVAL_LASTTHISMONTH` (Current and previous calendar months)
- `INTERVAL_THISNEXTMONTH` (Current and next calendar months)
- `INTERVAL_CURRENTQ` (Current calendar quarter)
- `INTERVAL_CURNEXTQ` (Current and next calendar quarters)
- `INTERVAL_CURPREVQ` (Current and previous calendar quarters)
- `INTERVAL_NEXTQ` (Next calendar quarter)
- `INTERVAL_PREVQ` (Previous calendar quarter)
- `INTERVAL_CURNEXT3Q` (Current and next three calendar quarters)
- `INTERVAL_CURY` (Current calendar year)
- `INTERVAL_PREVY` (Previous calendar year)
- `INTERVAL_PREV2Y` (Previous two calendar years)
- `INTERVAL_AGO2Y` (Two calendar years ago)
- `INTERVAL_NEXTY` (Next calendar year)
- `INTERVAL_PREVCURY` (Current and previous calendar years)
- `INTERVAL_PREVCUR2Y` (Current and previous two calendar years)
- `INTERVAL_CURNEXTY` (Current and next calendar years)

**Relative Date Intervals (Days):**

- `INTERVAL_LAST7` (Last 7 days)
- `INTERVAL_LAST30` (Last 30 days)
- `INTERVAL_LAST60` (Last 60 days)
- `INTERVAL_LAST90` (Last 90 days)
- `INTERVAL_LAST120` (Last 120 days)
- `INTERVAL_NEXT7` (Next 7 days)
- `INTERVAL_NEXT30` (Next 30 days)
- `INTERVAL_NEXT60` (Next 60 days)
- `INTERVAL_NEXT90` (Next 90 days)
- `INTERVAL_NEXT120` (Next 120 days)

**Entitlement Period Intervals (If applicable and enabled):**

- `CURRENT_ENTITLEMENT_PERIOD`
- `PREVIOUS_ENTITLEMENT_PERIOD`
- `PREVIOUS_TWO_ENTITLEMENT_PERIODS`
- `TWO_ENTITLEMENT_PERIODS_AGO`
- `CURRENT_AND_PREVIOUS_ENTITLEMENT_PERIOD`
- `CURRENT_AND_PREVIOUS_TWO_ENTITLEMENT_PERIODS`

_(Note: This list is derived from the provided source material. Always refer to the latest Salesforce Metadata API Developer Guide for the most up-to-date and comprehensive list of valid `UserDateInterval` values.)_

**Example Usage in a Report:**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <name>Opportunities Closing This Fiscal Quarter</name>
    <format>Summary</format>
    <reportType>Opportunity</reportType>
    <!-- ... columns, groupings ... -->
    <filter>
        <criteriaItems>
            <column>Opportunity.IsWon</column>
            <operator>equals</operator>
            <value>false</value>
        </criteriaItems>
    </filter>
    <timeFrameFilter>
        <dateColumn>Opportunity.CloseDate</dateColumn>
        <interval>INTERVAL_CURRENT</interval> <!-- Corresponds to "Current FQ" in UI based on fiscal year settings -->
    </timeFrameFilter>
</Report>
```

## 6. Defining Bucket Fields (`<buckets>`)

Bucket fields allow you to categorize report records on-the-fly without creating custom formula fields on the object itself. You can group related values in text, numeric, or picklist fields into named "buckets".

The `<buckets>` element contains one or more individual bucket field definitions. Each definition corresponds to the `ReportBucketField` type.

**Structure (within the main `<Report>` element):**

```xml
<buckets>
    <bucketType>number</bucketType> <!-- Type: number, text, or picklist -->
    <developerName>BucketField_MyNumericBucket</developerName> <!-- Required: Unique API name -->
    <masterLabel>My Numeric Bucket</masterLabel> <!-- Required: Display label -->
    <nullTreatment>z</nullTreatment> <!-- Optional: For numeric - 'z' (treat null as 0), 'n' (treat null as null) -->
    <otherBucketLabel>Other</otherBucketLabel> <!-- Optional: Label for values not falling into any bucket -->
    <sourceColumnName>Opportunity.Amount</sourceColumnName> <!-- Required: The field being bucketed -->
    <!-- One or more <values> elements defining each bucket range/value -->
    <values>
      <sourceValues>
        <from>0</from> <!-- Optional: Lower bound for numeric (exclusive) -->
        <to>5000</to>   <!-- Optional: Upper bound for numeric (inclusive) -->
      </sourceValues>
      <value>Small</value> <!-- Required: Name/label for this specific bucket -->
    </values>
    <values>
      <sourceValues>
        <from>5000</from>
        <to>25000</to>
      </sourceValues>
      <value>Medium</value>
    </values>
    <values>
      <sourceValues>
         <sourceValue>Specific Text Value</sourceValue> <!-- Used for bucketType 'text' or 'picklist' -->
      </sourceValues>
       <sourceValues>
         <sourceValue>Another Text Value</sourceValue>
      </sourceValues>
      <value>Specific Category</value>
    </values>
    <!-- ... more values ... -->
</buckets>
```

**Key Sub-Elements of `<buckets>` (type `ReportBucketField`):**

- **`<bucketType>`** (`ReportBucketFieldType`, Required): Specifies the type of bucket. Valid values: `text`, `number`, `picklist`.
- **`<developerName>`** (`string`, Required): A unique API name for this bucket field (e.g., `BucketField_Region`, `BucketField_DealSize`). Must follow standard API naming conventions.
- **`<masterLabel>`** (`string`, Required): The display label for the bucket field in the report builder and report (max 40 chars, spacing rules apply).
- **`<nullTreatment>`** (`ReportBucketFieldNullTreatment`, Optional): For numeric buckets only. Defines how empty/null values are handled. Valid values: `z` (treat as zero), `n` (treat as null/blank).
- **`<otherBucketLabel>`** (`string`, Optional): The label for the container holding values that don't fit into any defined bucket.
- **`<sourceColumnName>`** (`string`, Required): The API name of the field whose values are being categorized by this bucket field (e.g., `Account.Industry`, `Opportunity.Amount`).
- **`<values>`** (`ReportBucketFieldValue[]`): Represents a single bucket definition. A `ReportBucketField` typically contains multiple `<values>` elements.
  - **`<value>`** (`string`, Required): The name assigned to this specific bucket (e.g., "Small", "Large", "West Region").
  - **`<sourceValues>`** (`ReportBucketFieldSourceValue[]`, Required): Defines the criteria for source field values to fall into _this_ bucket.
    - **For `bucketType` 'number':** Use `<from>` (lower bound, non-inclusive) and/or `<to>` (upper bound, inclusive). The first numeric bucket value must only have `<to>`, the last must only have `<from>`, and all others must have both.
    - **For `bucketType` 'text' or 'picklist':** Use one or more `<sourceValue>` elements, each containing the exact text or picklist value from the source field that should be included in this bucket.

## 7. Defining Row-Level Formulas (`<reportCustomDetailFormula>`)

Row-level formulas perform calculations on each individual record (row) displayed in the report. They differ from Custom Summary Formulas (`<aggregates>`), which operate on summarized data (subtotals and grand totals). The result appears as a new column in the report.

The `CustomDetailFormulas` type defines these formulas. They are placed directly within the main `<Report>` element.

**Structure (within the main `<Report>` element):**

```xml
<reportCustomDetailFormula> <!-- Note: Source uses this singular tag, implies one definition per tag -->
    <calculatedFormula>Opportunity.Amount * Opportunity.Probability</calculatedFormula> <!-- Required: Formula logic using fields available in the row -->
    <dataType>Double</dataType> <!-- Required: Data type of the result -->
    <developerName>Expected_Value_RLF</developerName> <!-- Required: Unique internal name -->
    <label>Expected Value (RLF)</label> <!-- Required: Column header label -->
    <scale>2</scale> <!-- Required: Decimal places (0-18) -->
    <description>Calculates Amount * Probability for each opportunity row.</description> <!-- Optional -->
</reportCustomDetailFormula>
```

**Key Sub-Elements (type `CustomDetailFormulas`):**

- **`<calculatedFormula>`** (`string`, Required): The formula logic, referencing fields available at the row level for the report type.
- **`<dataType>`** (`ReportCustomDetailFormulaDatatype`, Required): Specifies the data type for formatting and display. Valid values: `Double`, `DateOnly`, `DateTime`, `Text`.
- **`<developerName>`** (`string`, Required): The unique internal development name for the formula (e.g., `FORMULA1`).
- **`<label>`** (`string`, Required): The name/label that identifies this formula, used as the column header.
- **`<scale>`** (`int`, Required): The number of decimal places for the formula result (valid values 0 through 18).
- **`<description>`** (`string`, Optional): A description for the formula (max 255 characters).

## 8. Defining Historical Trending Filters and Display

Salesforce allows tracking changes to key fields over time (Historical Trending). Report metadata includes elements to filter reports based on historical dates and control how historical data and changes are displayed.

**Key Elements for Historical Trending:**

- **`<historicalSelector>` (`ReportHistoricalSelector`)**: Defines the date range for which historical data is captured or displayed in the report. Placed directly within the main `<Report>` element.
- **`<snapshot>` attribute within `<criteriaItems>`**: Used in standard filters (`<filter>`) to compare a field's value against its value on a specific historical date.
- **Attributes within `<columns>`**: Controls the display of changes and colors in historical trend reports.

**Structure of `<historicalSelector>`:**

```xml
<historicalSelector>
    <snapshot>N_DAYS_AGO:90</snapshot> <!-- Required: Represents the historical date point or range -->
    <!-- Default is "Any Historical Date" if element is omitted or snapshot is unspecified -->
</historicalSelector>
```

- **`<snapshot>`** (`string`, Required within `historicalSelector`):
  - Represents the date value to apply as the historical filter point.
  - Format can be relative (e.g., `N_DAYS_AGO:7`, `N_MONTHS_AGO:3`) or absolute (`yyyy-MM-dd`).
  - If unspecified within the selector, it implies all historical data points might be considered (equivalent to "Any Historical Date").
  - Available in API version 29.0 and later.

**Historical Filtering using `<filter>`:**

Within a `<criteriaItems>` element of a standard `<filter>`, you can use the `<snapshot>` sub-element to compare a field's current value to its historical value or filter based on a historical value itself.

```xml
<filter>
    <criteriaItems>
        <column>Opportunity.Amount_hst</column> <!-- Note: Historical fields often have '_hst' suffix -->
        <operator>lessThan</operator>
        <snapshot>N_DAYS_AGO:30</snapshot> <!-- Comparing current value to value 30 days ago -->
        <value>FIELD_VALUE_ON_SNAPSHOT_DATE</value> <!-- Placeholder - actual comparison logic may vary -->
    </criteriaItems>
    <criteriaItems>
         <column>Opportunity.StageName_hst</column>
         <operator>equals</operator>
         <snapshot>2023-10-01</snapshot> <!-- Filter based on the value StageName had on Oct 1, 2023 -->
         <value>Prospecting</value>
    </criteriaItems>
</filter>
```

**Historical Display Options in `<columns>`:**

For reports based on historical trending objects, specific attributes within the `<columns>` element (type `ReportColumn`) control display:

- **`<reverseColors>`** (`boolean`, API v29.0+): Displays greater Date values as green and greater Amount values as red, reversing the default colors. Default `false`.
- **`<showChanges>`** (`boolean`, API v29.0+): Adds a column displaying the difference between the current and the historical Date and Amount values. Default `false`.
- **`<showCurrentDate>`** (`boolean`, API v29.0+): Specific to Matrix reports with historical trending, can be set to `true` to explicitly show the current date column alongside historical snapshots.

**Example Snippet (Conceptual):**

```xml
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <name>Opportunity Amount Changes - Last 90 Days</name>
    <reportType>OpportunityFieldHistory</reportType> <!-- Example RT -->
    <format>Summary</format>

    <historicalSelector>
      <snapshot>N_DAYS_AGO:90</snapshot>
    </historicalSelector>

    <columns>
        <field>OPPORTUNITY.NAME</field>
    </columns>
    <columns>
        <field>AMOUNT_hst</field> <!-- Historical Amount column -->
        <showChanges>true</showChanges> <!-- Show the change column -->
        <reverseColors>false</reverseColors>
    </columns>

    <!-- Filters, Groupings etc. -->
</Report>
```
