# Salesforce Report Metadata: Full Examples (report_full_examples.md)

## 1. Introduction

This document provides complete examples of Salesforce Report metadata (`.report-meta.xml` files or their core content). These examples illustrate how different components come together to define various report types and features. Annotations are provided to highlight key sections.

_(Note: These examples are synthesized based on the provided documentation snippets and structure. Specific field names like `CRT_Object_c` are illustrative placeholders from the source material.)_

## 2. Example 1: Matrix Report with Custom Summary Formulas and Buckets

This example showcases a Matrix report that uses row and column groupings, bucket fields for categorization, custom summary formulas for advanced calculations, and a chart.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Custom Summary Formula 1: Using PREVGROUPVAL and PARENTGROUPVAL -->
    <aggregates>
        <acrossGroupingContext>CRT_Object_c$Id</acrossGroupingContext>
        <calculatedFormula>PREVGROUPVAL(CRT_Object_c.Currency_c:AVG, CRT_Object_c.Id) - PARENTGROUPVAL(CRT_Object_c.Number_c:MAX, CRT_Object_c.CreatedBy.Name, COLUMN_GRAND_SUMMARY)/RowCount</calculatedFormula>
        <datatype>number</datatype>
        <developerName>FORMULA1</developerName>
        <downGroupingContext>CRT_Object_c$CreatedBy</downGroupingContext>
        <isActive>true</isActive>
        <masterLabel>CurrCSF</masterLabel>
        <scale>2</scale>
    </aggregates>
    <!-- Custom Summary Formula 2: Using IF, BLANKVALUE, ROUND -->
    <aggregates>
        <acrossGroupingContext>CRT_Object_c$LastModifiedDate</acrossGroupingContext>
        <calculatedFormula>IF(RowCount&gt;10, BLANKVALUE(ROUND(PREVGROUPVAL(CRT_Object_c.Currency_c:SUM, CRT_Object_c.LastModifiedDate),3), PARENTGROUPVAL(CRT_Object_c.Number_c:SUM, ROW_GRAND_SUMMARY, CRT_Object_c.Id)), 1000)</calculatedFormula>
        <datatype>number</datatype>
        <developerName>FORMULA2</developerName>
        <downGroupingContext>GRAND_SUMMARY</downGroupingContext>
        <isActive>true</isActive>
        <masterLabel>numCSF</masterLabel>
        <scale>2</scale>
    </aggregates>

    <!-- Bucket Field: Numeric Bucket for Sales Amount -->
    <buckets>
        <bucketType>number</bucketType>
        <developerName>BucketField_BusinessSize</developerName>
        <masterLabel>NumericBucket</masterLabel>
        <nullTreatment>z</nullTreatment> <!-- Treat nulls as zero -->
        <sourceColumnName>SALES</sourceColumnName> <!-- Assuming 'SALES' is a field on CRT_Object_c -->
        <values>
            <sourceValues>
                <to>10000</to>
            </sourceValues>
            <value>low</value>
        </values>
        <values>
            <sourceValues>
                <from>10000</from>
                <to>25000</to>
            </sourceValues>
            <value>mid</value>
        </values>
        <values>
            <sourceValues>
                <from>25000</from>
            </sourceValues>
            <value>high</value>
        </values>
    </buckets>
    <!-- Bucket Field: Text Bucket for Region based on State -->
    <buckets>
        <bucketType>text</bucketType>
        <developerName>BucketField_Region</developerName>
        <masterLabel>TextBucket</masterLabel>
        <nullTreatment>n</nullTreatment> <!-- Treat nulls as null -->
        <otherBucketLabel>Other</otherBucketLabel>
        <sourceColumnName>ADDRESS1_STATE</sourceColumnName> <!-- Assuming 'ADDRESS1_STATE' is a field -->
        <values>
            <sourceValues>
                <sourceValue>CA</sourceValue>
            </sourceValues>
            <value>west</value>
        </values>
        <values>
            <sourceValues>
                <sourceValue>NY</sourceValue>
            </sourceValues>
            <sourceValues>
                <sourceValue>Ontario</sourceValue>
            </sourceValues>
            <value>east</value>
        </values>
    </buckets>

    <!-- Chart Definition -->
    <chart>
        <backgroundColor1>#FFFFFF</backgroundColor1>
        <backgroundColor2>#FFFFFF</backgroundColor2>
        <backgroundFadeDir>Diagonal</backgroundFadeDir>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>FORMULA1</column> <!-- Plotting CSF 1 -->
        </chartSummaries>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>FORMULA2</column> <!-- Plotting CSF 2 -->
        </chartSummaries>
        <chartSummaries>
            <aggregate>Maximum</aggregate>
            <axisBinding>y</axisBinding>
            <column>CRT_Object_c$Number_c</column> <!-- Plotting a standard field aggregate -->
        </chartSummaries>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>RowCount</column> <!-- Plotting RowCount -->
        </chartSummaries>
        <chartType>VerticalColumn</chartType>
        <enableHoverLabels>true</enableHoverLabels>
        <expandOthers>true</expandOthers>
        <groupingColumn>CRT_Object_c$LastModifiedDate</groupingColumn> <!-- Chart grouping -->
        <legendPosition>Right</legendPosition>
        <location>CHART_TOP</location>
        <showAxisLabels>true</showAxisLabels>
        <size>Medium</size>
        <summaryAxisRange>Auto</summaryAxisRange>
        <textColor>#000000</textColor>
        <textSize>12</textSize>
        <titleColor>#000000</titleColor>
        <title>Sales Performance Analysis</title>
        <titleSize>18</titleSize>
    </chart>

    <!-- Column Definitions -->
    <columns>
        <field>CRT_Object_c$Name</field>
    </columns>
    <columns>
        <aggregateTypes>Average</aggregateTypes>
        <field>CRT_Object_c$Currency_c</field>
    </columns>
    <columns>
        <aggregateTypes>Maximum</aggregateTypes>
        <field>CRT_Object_c$Number_c</field>
    </columns>
    <columns>
        <field>BucketField_Region</field> <!-- Displaying bucket field -->
    </columns>
    <columns>
        <field>BucketField_BusinessSize</field> <!-- Displaying bucket field -->
    </columns>

    <format>Matrix</format>

    <!-- Column Groupings (Across) -->
    <groupingsAcross>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object_c$Id</field> <!-- Unusual to group by ID, but per source example -->
        <sortOrder>Asc</sortOrder>
    </groupingsAcross>
    <groupingsAcross>
        <dateGranularity>Year</dateGranularity>
        <field>CRT_Object_c$LastModifiedDate</field>
        <sortOrder>Asc</sortOrder>
    </groupingsAcross>

    <!-- Row Groupings (Down) -->
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object_c$CreatedBy</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object_c$Currency_c</field> <!-- Grouping by a currency field (granularity Day might be less meaningful here) -->
        <sortOrder>Desc</sortOrder>
    </groupingsDown>

    <name>CrtMMVC</name> <!-- Report Name -->
    <reportType>CRT1_c</reportType> <!-- Assuming CRT1_c is a custom report type -->
    <scope>organization</scope>
    <showDetails>false</showDetails> <!-- Collapsed view, showing summaries -->

    <!-- Time Frame Filter -->
    <timeFrameFilter>
        <dateColumn>CRT_Object_c$CreatedDate</dateColumn>
        <interval>INTERVAL_CUSTOM</interval> <!-- Implies startDate and endDate would be used in a full UI setup or specific API call -->
        <!-- <startDate>YYYY-MM-DD</startDate> -->
        <!-- <endDate>YYYY-MM-DD</endDate> -->
    </timeFrameFilter>
</Report>
```

## 3. Example 2: Joined Report with Cross-Block and Standard Custom Summary Formulas

This example illustrates a Joined Report. It includes two blocks, one for Opportunities and another for Accounts (though the second block's content is minimal in this snippet). It demonstrates a cross-block CSF and a standard CSF specific to a block.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <!-- Cross-Block Custom Summary Formula -->
    <aggregates>
        <calculatedFormula>B1#AMOUNT:SUM+B2#EMPLOYEES:SUM</calculatedFormula> <!-- References Block B1 and B2 -->
        <datatype>number</datatype>
        <developerName>FORMULA</developerName>
        <isActive>true</isActive>
        <isCrossBlock>true</isCrossBlock>
        <masterLabel>Cross-Block CSF Example</masterLabel>
        <scale>2</scale>
    </aggregates>
    <!-- Standard Custom Summary Formula (for Opportunity block) -->
    <aggregates>
        <calculatedFormula>AMOUNT:SUM</calculatedFormula> <!-- Specific to the Opportunity block -->
        <developerName>FORMULA2</developerName>
        <isActive>true</isActive>
        <isCrossBlock>false</isCrossBlock>
        <masterLabel>Standard CSF Example</masterLabel>
        <reportType>Opportunity</reportType> <!-- Associates with Opportunity block -->
        <scale>2</scale>
    </aggregates>

    <!-- Block 1: Opportunities -->
    <block>
        <blockInfo>
            <aggregateReferences>
                <aggregate>FORMULA2</aggregate> <!-- Referencing the standard CSF -->
            </aggregateReferences>
            <blockId>B1</blockId>
            <joinTable>a</joinTable> <!-- 'a' likely refers to the primary object being joined on, often Account -->
        </blockInfo>
        <columns>
            <field>TYPE</field>
        </columns>
        <columns>
            <field>OPPORTUNITY_NAME</field>
        </columns>
        <columns>
            <aggregateTypes>Sum</aggregateTypes>
            <field>AMOUNT</field>
        </columns>
        <format>Summary</format>
        <name>Opportunities Block 1</name>
        <!-- Parameters for Opportunity block -->
        <params>
            <name>probability</name>
            <value>0</value> <!-- Example parameter: probability greater than or equal to 0 -->
        </params>
        <params>
            <name>open</name>
            <value>all</value> <!-- Example parameter: show all open/closed statuses -->
        </params>
        <reportType>Opportunity</reportType>
        <scope>organization</scope>
        <timeFrameFilter>
            <dateColumn>CLOSE_DATE</dateColumn>
            <interval>INTERVAL_CUSTOM</interval>
        </timeFrameFilter>
    </block>

    <!-- Block 2: Accounts (Simplified for example) -->
    <block>
        <blockInfo>
            <aggregateReferences>
                <!-- <aggregate>FORMULA</aggregate> --> <!-- Could reference the cross-block CSF here or in chart -->
            </aggregateReferences>
            <blockId>B2</blockId>
            <joinTable>a</joinTable>
        </blockInfo>
        <columns>
            <field>USERS.NAME</field> <!-- User who owns the account -->
        </columns>
        <columns>
            <field>ACCOUNT_TYPE</field>
        </columns>
        <columns>
            <aggregateTypes>Sum</aggregateTypes>
            <field>EMPLOYEES</field>
        </columns>
        <format>Summary</format>
        <name>Accounts block 2</name>
        <reportType>AccountList</reportType> <!-- Or simply 'Account' depending on available report types -->
        <scope>organization</scope>
        <timeFrameFilter>
            <dateColumn>CREATED_DATE</dateColumn>
            <interval>INTERVAL_CUSTOM</interval>
        </timeFrameFilter>
    </block>

    <!-- General Block Info for the Joined Report Structure -->
    <blockInfo>
        <blockId xsi:nil="true"/> <!-- Root blockInfo, often nil -->
        <joinTable>a</joinTable>
    </blockInfo>

    <!-- Chart for Joined Report (referencing cross-block CSF or block-specific data) -->
    <chart>
        <backgroundColor1>#FFFFFF</backgroundColor1>
        <backgroundColor2>#FFFFFF</backgroundColor2>
        <backgroundFadeDir>Diagonal</backgroundFadeDir>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>B1#RowCount</column> <!-- Charting RowCount from Block 1 -->
        </chartSummaries>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>FORMULA</column> <!-- Charting the cross-block CSF -->
        </chartSummaries>
        <chartType>HorizontalBar</chartType>
        <enableHoverLabels>false</enableHoverLabels>
        <expandOthers>true</expandOthers>
        <groupingColumn>ACCOUNT_NAME</groupingColumn> <!-- Grouping by Account Name (common across blocks) -->
        <location>CHART_TOP</location>
        <showAxisLabels>true</showAxisLabels>
        <size>Medium</size>
        <summaryAxisRange>Auto</summaryAxisRange>
        <textColor>#000000</textColor>
        <textSize>12</textSize>
        <titleColor>#000000</titleColor>
        <title>Joined Report: Opps and Accounts Summary</title>
        <titleSize>18</titleSize>
    </chart>

    <format>MultiBlock</format> <!-- Format for Joined Reports -->
    <!-- Row Groupings for the Joined Report (applies to the overall joined view) -->
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>ACCOUNT_NAME</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>

    <name>Joined Opps and Accounts</name>
    <reportType>Opportunity</reportType> <!-- Primary report type for the joined report can be one of the block types -->
    <showDetails>true</showDetails>
</Report>
```

## 4. Example 3: Sample Report with groups and filters

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
    <aggregates>
        <acrossGroupingContext>CRT_Object__c$Id</acrossGroupingContext>
        <calculatedFormula>PREVGROUPVAL(CRT_Object__c.Currency__c:AVG, CRT_Object__c.Id) *
                PARENTGROUPVAL(CRT_Object__c.Number__c:MAX, CRT_Object__c.CreatedBy.Name,
                COLUMN_GRAND_SUMMARY)/RowCount</calculatedFormula>
        <datatype>number</datatype>
        <developerName>FORMULA1</developerName>
        <downGroupingContext>CRT_Object__c$CreatedBy</downGroupingContext>
        <isActive>true</isActive>
        <masterLabel>CurrCSF</masterLabel>
        <scale>2</scale>
    </aggregates>
    <aggregates>
        <acrossGroupingContext>CRT_Object__c$LastModifiedDate</acrossGroupingContext>
        <calculatedFormula>IF(RowCount&gt;10,
                BLANKVALUE(ROUND(PREVGROUPVAL(CRT_Object__c.Currency__c:SUM,
                CRT_Object__c.LastModifiedDate),3),
                PARENTGROUPVAL(CRT_Object__c.Number__c:SUM, ROW_GRAND_SUMMARY,
                CRT_Object__c.Id))  , 1000)</calculatedFormula>
        <datatype>number</datatype>
        <developerName>FORMULA2</developerName>
        <downGroupingContext>GRAND_SUMMARY</downGroupingContext>
        <isActive>true</isActive>
        <masterLabel>numCSF</masterLabel>
        <scale>2</scale>
    </aggregates>
    <buckets>
        <bucketType>number</bucketType>
        <developerName>BucketField_BusinessSize</developerName>
        <masterLabel>NumericBucket</masterLabel>
        <nullTreatment>z</nullTreatment>
        <sourceColumnName>SALES</sourceColumnName>
        <values>
            <sourceValues>
                <to>10000</to>
            </sourceValues>
            <value>low</value>
        </values>
        <values>
            <sourceValues>
                <from>10000</from>
                <to>25000</to>
            </sourceValues>
            <value>mid</value>
        </values>
        <values>
            <sourceValues>
                <from>25000</from>
            </sourceValues>
            <value>high</value>
        </values>
    </buckets>
    <buckets>
        <bucketType>text</bucketType>
        <developerName>BucketField_Region</developerName>
        <masterLabel>TextBucket</masterLabel>
        <nullTreatment>n</nullTreatment>
        <otherBucketLabel>Other</otherBucketLabel>
        <sourceColumnName>ADDRESS1_STATE</sourceColumnName>
        <values>
            <sourceValues>
                <sourceValue>CA</sourceValue>
            </sourceValues>
            <value>west</value>
        </values>
        <values>
            <sourceValues>
                <sourceValue>NY</sourceValue>
            </sourceValues>
            <sourceValues>
                <sourceValue>Ontario</sourceValue>
            </sourceValues>
            <value>east</value>
        </values>
    </buckets>
    <chart>
        <backgroundColor1>#FFFFFF</backgroundColor1>
        <backgroundColor2>#FFFFFF</backgroundColor2>
        <backgroundFadeDir>Diagonal</backgroundFadeDir>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>FORMULA1</column>
        </chartSummaries>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>FORMULA2</column>
        </chartSummaries>
        <chartSummaries>
            <aggregate>Maximum</aggregate>
            <axisBinding>y</axisBinding>
            <column>CRT_Object__c$Number__c</column>
        </chartSummaries>
        <chartSummaries>
            <axisBinding>y</axisBinding>
            <column>RowCount</column>
        </chartSummaries>
        <chartType>VerticalColumn</chartType>
        <groupingColumn>CRT_Object__c$LastModifiedDate</groupingColumn>
        <legendPosition>Right</legendPosition>
        <location>CHART_TOP</location>
        <size>Medium</size>
        <summaryAxisRange>Auto</summaryAxisRange>
        <textColor>#000000</textColor>
        <textSize>12</textSize>
        <titleColor>#000000</titleColor>
        <titleSize>18</titleSize>
    </chart>
    <columns>
        <field>CRT_Object__c$Name</field>
    </columns>
    <columns>
        <aggregateTypes>Average</aggregateTypes>
        <field>CRT_Object__c$Currency__c</field>
    </columns>
    <columns>
        <aggregateTypes>Maximum</aggregateTypes>
        <field>CRT_Object__c$Number__c</field>
    </columns>
    <columns>
        <field>BucketField__Region</field>
    </columns>
    <format>Matrix</format>
    <groupingsAcross>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object__c$Id</field>
        <sortOrder>Asc</sortOrder>
    </groupingsAcross>
    <groupingsAcross>
        <dateGranularity>Year</dateGranularity>
        <field>CRT_Object__c$LastModifiedDate</field>
        <sortOrder>Asc</sortOrder>
    </groupingsAcross>
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object__c$CreatedBy</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>CRT_Object__c$Currency__c</field>
        <sortOrder>Desc</sortOrder>
    </groupingsDown>
    <name>CrtMMVC</name>
    <reportType>CRT1__c</reportType>
    <scope>organization</scope>
    <showDetails>false</showDetails>
    <timeFrameFilter>
        <dateColumn>CRT_Object__c$CreatedDate</dateColumn>
        <interval>INTERVAL_CUSTOM</interval>
    </timeFrameFilter>
</Report>
```

## 5. Example 4: Joined report

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Report xmlns="http://soap.sforce.com/2006/04/metadata">
<!-- This is a cross-block custom summary formula. Note that the calculated formula reference for a blocks reference uses the BlockId#Aggregate. -->
    <aggregates>
        <calculatedFormula>B1#AMOUNT:SUM+B2#EMPLOYEES:SUM</calculatedFormula>
        <datatype>number</datatype>
        <developerName>FORMULA</developerName>
        <isActive>true</isActive>
        <isCrossBlock>true</isCrossBlock>
        <masterLabel>Cross-Block CSF Example</masterLabel>
        <scale>2</scale>
    </aggregates>
<!-- This is a standard custom summary formula. Note that the calculated formula reference does not have block reference but just the aggregate name of the report type associated (Opportunity).-->
    <aggregates>
        <calculatedFormula>AMOUNT:SUM</calculatedFormula>
        <developerName>FORMULA2</developerName>
        <isActive>true</isActive>
        <isCrossBlock>false</isCrossBlock>
        <masterLabel>Standard CSF Example</masterLabel>
        <reportType>Opportunity</reportType>
        <scale>2</scale>
    </aggregates>
    <block>
      <blockInfo>
<!-- This is how the block defines that the custom summary formula should be referenced. In this example, it’s the in standard FORMULA 2 defined above. This block report has blockID B1.-->
        <aggregateReferences>
          <aggregate>FORMULA2</aggregate>
        </aggregateReference>
        <blockId>B1</blockId>
        <joinTable>a</joinTable>
      </blockInfo>
      <columns>
        <field>TYPE</field>
      </columns>
      <format>Summary</format>
      <name>Opportunities BLock 3</name>
      <params>
        <name>role_territory</name>
        <value>role</value>
      </params>
      <params>
        <name>terr</name>
        <value>all</value>
      </params>
      <params>
        <name>open</name>
        <value>all</value>
      </params>
      <params>
        <name>probability</name>
        <value>0</value>
      </params>
      <params>
        <name>co</name>
        <value>1</value>
      </params>
      <reportType>Opportunity</reportType>
      <scope>organization</scope>
      <timeFrameFilter>
        <dateColumn>CLOSE_DATE</dateColumn>
        <interval>INTERVAL_LASTTHISMONTH</interval>
      </timeFrameFilter>
    </block>
    <block>
      <blockInfo>
<!-- This is how the block defines that the custom summary formula should be referenced. In this example, it’s the cross-block custom summary formula FORMULA 1 defined above. This block report has blockId B2.-->
        <aggregateReferences>
          <aggregate>FORMULA1</aggregate>
        </aggregateReferences>
        <blockId>B2</blockId>
        <joinTable>a</joinTable>
      </blockInfo>
      <columns>
        <field>USERS.NAME</field>
      </columns>
      <columns>
        <field>TYPE</field>
      </columns>
      <columns>
         <field>DUE_DATE</field>
      </columns>
      <columns>
        <field>LAST_UPDATE</field>
      </columns>
      <columns>
        <field>ADDRESS1_STATE</field>
      </columns>
      <format>Summary</format
      <name>Accounts block 5</name>
      <params>
        <name>terr</name>
        <value>all</value>
      </params>
      <params>
        <name>co</name>
        <value>1</value>
      </params>
      <reportType>AccountList</reportType>
      <scope>organization</scope>
      <timeFrameFilter>
        <dateColumn>CREATED_DATE</dateColumn>
        <interval>INTERVAL_CUSTOM</interval>
      </timeFrameFilter>
    </block>
    <blockInfo>
      <blockId xsi:nil="true"/>
      <joinTable>a</joinTable>
    </blockInfo>
<chart>
        <backgroundColor1>#FFFFFF</backgroundColor1>
        <backgroundColor2>#FFFFFF</backgroundColor2>
        <backgroundFadeDir>Diagonal</backgroundFadeDir>
        <chartSummaries>
            <axisBinding>y</axisBinding>
<!-- This is how chart aggregates are designed in multiblock. We're using RowCount from Block 1.-->
            <column>B1#RowCount</column>
        </chartSummaries>
        <chartType>HorizontalBar</chartType>
        <enableHoverLabels>false</enableHoverLabels>
        <expandOthers>true</expandOthers>
        <groupingColumn>ACCOUNT_NAME</groupingColumn>
        <location>CHART_TOP</location>
        <showAxisLabels>true</showAxisLabels>
        <showPercentage>false</showPercentage>
        <showTotal>false</showTotal>
        <showValues>false</showValues>
        <size>Medium</size>
        <summaryAxisRange>Auto</summaryAxisRange>
        <textColor>#000000</textColor>
        <textSize>12</textSize>
        <titleColor>#000000</titleColor>
        <titleSize>18</titleSize>
    </chart>
    <format>MultiBlock</format>
    <groupingsDown>
        <dateGranularity>Day</dateGranularity>
        <field>ACCOUNT_NAME</field>
        <sortOrder>Asc</sortOrder>
    </groupingsDown>
    <name>mb_mbapi</name>
    <reportType>Opportunity</reportType>
    <showDetails>true</showDetails>
</Report>
```
