<!-- machine_translated: true -->

<!-- pre-align:aligned sig=a160539b5f44 -->

<a id="foundry.console.guide"></a>

## Machine Learning > NHN Cloud Foundry > Console User Guide { #foundry.console.guide }

This document describes how to manage data sources, pipelines, analytics (queries, charts, and dashboards), and apps in the NHN Cloud Foundry console.

The Required column in settings tables indicates the following:

- `O`: Required field
- `X`: Optional field
- `O*`: Required or optional depending on other settings

<a id="status"></a>

## Status { #status }

Console path: **Machine Learning > NHN Cloud Foundry > Status** tab

The Status tab shows the service activation status and tenant settings. To use the service, check the activation status on this tab first. If the service is not activated, you must submit an activation request.

<a id="status.activate"></a>
### Request Service Activation { #status.activate }

Service activation cannot be performed directly in the console. Contact us via [1:1 Inquiry](https://www.nhncloud.com/kr/support/inquiry) and include the resource size you need. After the requested cluster is created, you can start using the service beginning with data source creation.

The features available for each resource size are as follows:

| Value | Description |
| --- | --- |
| SMALL | Basic resources: data sources, common features, and chart queries available |
| MEDIUM | Basic resources + data pipeline additionally available |
| LARGE | Basic resources + AI app available |
| XLARGE | All features available (data pipeline + AI app) |

<a id="status.info"></a>
### Check Service Status { #status.info }

After activation, you can check the following information on the Status tab:

| Item | Description |
| --- | --- |
| Service status | Current activation status of the service |
| Tenant domain | Domain used to access the service |

- If you need to change resources after activation, submit a request via 1:1 Inquiry.
- While the service environment is being configured or cleaned up, a progress indicator is displayed.
- Click the **Deactivate** button to deactivate the service.

!!! danger "Caution"
    Deactivating the service deletes all created resources, and this action cannot be undone.

<a id="datasource"></a>

## Data Source { #datasource }

Console path: **Machine Learning > NHN Cloud Foundry > Data Source** tab

A data source is a unit that stores data to be analyzed in NHN Cloud Foundry. You can create, view, and delete data sources from the console.

<a id="datasource.list"></a>
### Data Source List { #datasource.list }

You can view the following information on the data source list screen.

| Column | Description |
| --- | --- |
| Type | Type of the data source |
| Data source name | Name that identifies the data source |
| Table name | Name of the table where data is stored |
| Status | Current status of the data source |
| Data volume | Number of rows of ingested data |
| Created on | Date and time the data source was created |
| Details | Click the magnifying glass icon to view detailed information |

Type:

| Value | Description |
| --- | --- |
| File | Data source created from an uploaded CSV file |
| Recommendation | Data source where recommendation results are stored |
| Dataset | Data source created by a pipeline |

Status:

| Value | Description |
| --- | --- |
| INITIALIZING | Initializing data source |
| PROCESSING | Processing data |
| INGESTING | Ingesting data |
| COMPLETED | Data source processing complete |
| FAILED | Data source processing failed |
| DELETING | Deleting data source |

For data sources of the File type, the most recent file upload result is displayed as an icon next to the status.

| Value | Description |
| --- | --- |
| Applying | The most recently uploaded file is being applied |
| Review Recommended | The most recently uploaded file has been applied, but there are items to check. Review the details in the data source details view |
| Review Required | The data source is available, but the most recently uploaded file was not applied. Check the file and upload it again |

- Hover over the question mark icon next to the status badge to see a description of the status.
- Use the search feature at the top to filter by data source name or ID.
- You can also narrow results using the column header filters for the Type, Data source name, Table name, and Status columns.
- You can adjust the number of items displayed per page (10, 20, or 50; default is 10).

!!! tip "Note"
    Items of the Recommendation and Dataset types cannot be created manually. Recommendation data sources are created automatically when you create a recommendation system app, and Dataset data sources are created automatically when you run a pipeline.

<a id="datasource.create"></a>
### Create Data Source { #datasource.create }

Click the **Create Data Source** button to open the creation modal.

<a id="datasource.create.basic"></a>
#### Basic Settings { #datasource.create.basic }

| Item | Required | Description |
| --- | --- | --- |
| Data source name | O | Korean, Japanese, English, numbers, spaces, hyphens (-), and underscores (_) allowed (between 1 and 64 characters) |
| Table name | O | Lowercase English letters, numbers, and underscores (_) allowed (between 1 and 64 characters). Cannot start with a number or use SQL reserved words |
| Description | X | Data source description |

Data source names and table names that are already in use cannot be used.

<a id="datasource.create.detail"></a>
#### Detail Settings { #datasource.create.detail }

| Item | Required | Description |
| --- | --- | --- |
| CSV file | X | Select a CSV file to upload using the **Select File** button (up to 100 MB) |
| Header settings | X | When **First row is a header** is checked, the first row of the CSV is used as column names |
| Primary key field | X | Specify the column to identify rows using a dropdown |
| Schema | O | Define field names and types for the data |

Specify the primary key field as follows:

- When you select a CSV file, a dropdown is automatically created with the first column selected. Click the dropdown to change to a different column.
- Clicking **Add Primary Key** adds another dropdown, allowing you to specify a composite primary key with two or more columns. Columns already selected will not appear as candidates in other dropdowns.
- Clearing all dropdowns leaves the primary key unspecified.
- For files without headers, columns are displayed using their index number and the values from the first data row.

Enter the schema using one of the following methods:

- Type inference: After selecting a CSV file, click the **Infer Types** button to infer column types from a sample of the beginning of the CSV (up to 1,000 rows) and populate the schema input fields. You can manually correct any types that were inferred incorrectly.
- Table editing: Use the **Add Field** button to add rows and specify the field name and type for each row. The Remarks column shows items from the type inference results that require review.
- JSON editing: Click the **Edit as JSON** button and enter a JSON array in the format shown below.

Schema JSON format example:

```json
[
  { "fieldName": "hostname", "fieldType": "string" },
  { "fieldName": "portName", "fieldType": "string" },
  { "fieldName": "trafficIn", "fieldType": "double" },
  { "fieldName": "eventTimestamp", "fieldType": "timestamp" }
]
```

Supported field types:

| Value | Description |
| --- | --- |
| boolean | Boolean |
| int | Integer (32-bit) |
| long | Integer (64-bit) |
| float | Floating-point number (32-bit) |
| double | Floating-point number (64-bit) |
| string | String |
| timestamp | Timestamp |
| datetime | Date and time (YYYY-MM-DD HH:MM:SS) |
| date | Date |
| array | Array (default) |
| array&lt;double&gt; | Double array |
| array&lt;int&gt; | Integer array |
| array&lt;string&gt; | String array (for recommendation models) |
| array&lt;float&gt; | Float array (for recommendation models) |

!!! danger "Caution"
    The reserved field name `system_eventTimestamp` cannot be used.

After completing the settings, click the **Add** button to create the data source.

<a id="datasource.delete"></a>
### Delete Data Source { #datasource.delete }

Select the data sources to delete using the checkboxes in the list, then click the **Delete** button.

- When deleted, the table and all ingested data are also deleted.
- Data sources cannot be deleted while an ingestion job is in progress.

<a id="datasource.detail"></a>
### View Details / Preview { #datasource.detail }

- **View Details** (magnifying glass icon): View information about the data source.
- **Preview** (⌄ icon): Preview ingested data in table format.

The details view consists of the following tabs.

| Tab | Description |
| --- | --- |
| Connection Info | Data source ID, data source name, table name, type, description, status |
| Catalog | View field names and data type list, **Add Field** |

<a id="datasource.edit"></a>
### Update Data { #datasource.edit }

The data source name and table name cannot be changed after creation. Ingested data can be updated with a new CSV file, and fields can be added.

<a id="datasource.edit.csv"></a>
#### Edit Data with CSV { #datasource.edit.csv }

In the list, click the more options (⋯) menu on the row for a data source of the File type, then click **Edit with CSV**.

1. Review the existing column list.
2. Select a new CSV file using the **Select File** button (up to 100 MB).
3. Review the preview and checklist for the selected file.
4. Click the **Overwrite** button to start the upload. You can check the progress on the list screen.

The following situations restrict the upload:

| Situation | Guidance |
| --- | --- |
| The file has more columns than the existing schema | Add the required fields to the catalog first, then upload again |
| The file contains columns not in the catalog | Add the required fields to the catalog first, then upload again |
| The header setting does not match the first row of the file | Upload a file that has a column name row, or change the header settings of the data source |

<a id="datasource.edit.field"></a>
#### Add Fields { #datasource.edit.field }

Click the **Add Field** button on the **Catalog** tab in the View Details pane.

| Item | Required | Description |
| --- | --- | --- |
| Field name | O | Must start with a letter or underscore (_) and contain only letters, numbers, and underscores (_) |
| Type | O | Select from the field types supported in the schema |
| Description | X | Field description |

- You can add up to 20 fields at a time using the **Add Row** button.
- System-reserved field names and existing field names cannot be used.

!!! tip "Note"
    You can also update data through the API, in addition to the console. For more information, see the 'Ingest API' section in the [API Guide](../api-guide/#ingest.api).

<a id="pipeline"></a>

## Pipeline { #pipeline }

Console path: **Machine Learning > NHN Cloud Foundry > Pipeline** tab

A pipeline is a feature that processes data from a data source using a node-connected workflow to transform it into an analyzable dataset.

<a id="pipeline.list"></a>

### Pipeline List { #pipeline.list }

When you enter the Pipeline menu, the list of created pipelines is displayed in a table.

| Column | Description |
| --- | --- |
| Checkbox | Select a pipeline |
| Enabled | Icon indicating whether the pipeline is enabled |
| Pipeline name | Name that identifies the pipeline |
| Batch schedule | Configured schedule. Displayed as a hyphen if not configured. |
| Start date and time | Scheduled start date and time |
| End date and time | Scheduled end date and time |
| Last run date | Most recent run date and time |
| Pipeline status | Current status of the pipeline |
| Manual run | Run button |

- Click a row to go to the edit screen.
- For a running pipeline, a progress indicator appears in the Manual Run column, and the button is disabled when the pipeline cannot be run.

You can create, modify, delete, enable, and disable pipelines from the toolbar. Enable and Disable are available in the **More (⋯)** menu.

<a id="pipeline.create"></a>

### Create Pipeline { #pipeline.create }

1. On the list screen, click the **Create Pipeline** button.
2. In the **Settings** panel on the right, enter the basic information.

    | Item | Required | Description |
    | --- | --- | --- |
    | Pipeline name | O | Name that identifies the pipeline (up to 30 characters). Supports Korean, Japanese, English, numbers, spaces, hyphens (-), and underscores (_). |
    | Pipeline description | X | Additional description of the pipeline (up to 255 characters). |
    | Tag | X | Tags for categorization (up to 10 tags, up to 50 characters each). |

3. Click the **Create** button to create the pipeline in draft status.

!!! danger "Caution"
    If you save the settings of an enabled pipeline, the pipeline will be disabled. It will be automatically enabled when you click the Run button to build.

<a id="pipeline.editor"></a>

### Pipeline Editor { #pipeline.editor }

This is the main editing screen that you enter when creating or editing a pipeline.

- **Header**: Pipeline name, description, and back button
- **Tab bar (left)**: Run/stop, run history, and add source node buttons
- **Tab bar (right)**: Last run timestamp, status/build/active badges, and save button
- **Editor area**: Node-edge editor (drag-and-drop and auto-layout supported)
- **Side panel**: Settings, schedule, and computing resource panels

<a id="pipeline.status"></a>
#### Pipeline Status { #pipeline.status }

You can hover over the status badge to view a detailed description in a tooltip.

| Value | Description |
| --- | --- |
| Draft | The pipeline has been created and has no nodes |
| Modified | The pipeline configuration has changed and needs to be rebuilt (run again) |
| Building | The pipeline is being built |
| Waiting | The pipeline build is complete and ready to run |
| Running | The pipeline is running |
| Completed | The pipeline has completed successfully |
| Failed | The pipeline was stopped or an error occurred |
| Terminated | The user has stopped the pipeline run |
| Deleting | The pipeline is being deleted |

<a id="pipeline.node"></a>

### Node configuration { #pipeline.node }

A pipeline is configured by combining the following five node types.

| Node Type | Role | Input | Output | Description |
| --- | --- | --- | --- | --- |
| Source | Data origin | None | Data | Connects to an external data source |
| Transform | Data processing | Data | Transformed data | Filter, derived columns, aggregation, label encoding, and more |
| Join | Table joining | 2 inputs | Joined data | Inner/Left/Right/Full Outer/Semi/Anti joins |
| Union | Table merging | 2 inputs | Merged data | Full Merge/Intersect Merge/Left-First Merge |
| Dataset | Output table | Data | None | Saves results as a table (final node) |

Connection constraints:

- A SOURCE node cannot have inputs connected to it (root node).
- A DATASET node cannot have outputs connected to it (final node).
- Circular references (loops) or self-connections are not allowed.

!!! danger "Caution"
    The pipeline configuration is validated when saved. All nodes must be connected in a single flow (disconnected nodes are not allowed), and the last node in the flow must be exactly one DATASET node.

<a id="pipeline.node.source"></a>
#### Add source node { #pipeline.node.source }

1. Click the **Add Source Node** button in the tab bar.
2. A modal displays a list of available data sources.
3. Select the source you want to add.

| Value | Description |
| --- | --- |
| FILE | CSV file data source |
| Recommendation | Recommendation result store |
| Dataset | Data generated by a Dataset node in a pipeline. Can be reused as input for another pipeline. |

<a id="pipeline.node.transform"></a>
#### Transform node (TRANSFORM) { #pipeline.node.transform }

Each transformation requires a node name (up to 30 characters; Korean, Japanese, English, numbers, spaces, `-`, and `_` are allowed). Adding multiple transformations to a single node generates nodes connected in sequence.

The supported transformation methods are as follows:

| Category | Method | Description |
| --- | --- | --- |
| Row | Filter | Extracts only rows that match the specified condition |
| Row | Explode | Splits a delimited string into individual rows |
| Row | Derive | Creates derived columns using whitelisted functions |
| Aggregation | Aggregate | Groups and aggregates data |
| Window | Rank Top N | Global sort + top N rows |
| Column | Label Encode FIT | Creates a category-to-integer mapping table |
| Column | Label Encode Apply | Applies encoding using the mapping table |
| LLM | Classify | Classifies text into categories using an LLM |

<a id="pipeline.node.transform.filter"></a>
##### Filter { #pipeline.node.transform.filter }

| Value | Description | Example |
| --- | --- | --- |
| = | Equal to | field = 'value' |
| ≠ | Not equal to | field ≠ 'value' |
| > | Greater than | field > 100 |
| ≥ | Greater than or equal to | field ≥ 100 |
| &lt; | Less than | field < 100 |
| ≤ | Less than or equal to | field ≤ 100 |
| LIKE | Pattern matching | field LIKE '%keyword%' |
| IN | Included in list | field IN ('a', 'b', 'c') |
| IS NULL | Is NULL | field IS NULL |
| IS NOT NULL | Is not NULL | field IS NOT NULL |

Only the operators that are compatible with the selected column's type are displayed.

| Column Type | Available Operators |
| --- | --- |
| String | =, ≠, LIKE, IN, IS NULL, IS NOT NULL |
| Number | =, ≠, >, ≥, &lt;, ≤, IN, IS NULL, IS NOT NULL |
| Date/Time | =, ≠, >, ≥, &lt;, ≤, IS NULL, IS NOT NULL |
| Boolean | =, ≠, IS NULL, IS NOT NULL |

The value to compare against can be selected from: **Direct input**, **Time (relative to now)**, or **Another column**.

The logical operators `AND` and `OR` are supported. You can use the **Add Group** button to nest conditions up to three levels deep.

<a id="pipeline.node.transform.explode"></a>
##### Explode { #pipeline.node.transform.explode }

Splits a string column delimited by a separator into separate rows for each token.

| Item | Required | Description |
| --- | --- | --- |
| Column to split | O | The column containing delimited strings |
| Output column name | O | The name of the column to hold each split token |
| Delimiter | O | Default is a comma (,) |
| Trim whitespace | X | When ON, trims leading and trailing whitespace from each token |
| Remove empty tokens | X | When ON, excludes empty string tokens from the output rows |
| Output type | X | STRING, BIGINT, INT, DOUBLE. Defaults to STRING if not specified |
| Source columns to retain | X | Source columns to include in the output. If not selected, all source columns are retained |

- The delimiter is interpreted as a regular expression, so special characters must be escaped (for example, `\.`, `\|`).
- When an output type is specified, the split values are cast to that type. For example, if numeric IDs are comma-separated, specifying BIGINT allows you to match the join key type.

<a id="pipeline.node.transform.derive"></a>
##### Derived column (Derive) { #pipeline.node.transform.derive }

Creates new columns from existing columns using provided functions. You can add multiple definitions using the **Add Derived Column** button, specifying a function and a derived column name for each definition.

| Value | Description | Additional Settings |
| --- | --- | --- |
| DATE_BUCKET | Date bucket (epoch ms → date in specified timezone) | Source column, timezone (e.g., Asia/Seoul) |
| JSON_ARRAY_LENGTH | Number of elements in a JSON array | Source column, element schema |
| CONCAT | Concatenates multiple columns with a delimiter | 2 or more source columns, delimiter, default value (optional) |
| RATIO | Calculates the ratio of numerator to denominator | Numerator, denominator, multiplier (optional, default 1), rounding precision (optional) |
| COALESCE | Returns the first non-null value from multiple columns | 2 or more source columns |

- Definitions are applied in order from top to bottom. A derived column created by an earlier definition can be referenced as a source column in a later definition.
- Derived column names can contain English letters, numbers, underscores (_), and Korean characters.

<a id="pipeline.node.transform.aggregate"></a>
##### Aggregate { #pipeline.node.transform.aggregate }

1. **Select grouping criteria** (optional): Groups rows that share the same values. If not selected, all data is aggregated as a single group.
2. **Define aggregation functions** (required): Specify the column to aggregate, the aggregation function, and the output column name. You can add multiple definitions.

| Value | Description |
| --- | --- |
| COUNT | Count |
| COUNT_DISTINCT | Count of distinct values |
| SUM | Sum |
| AVG | Average |
| MIN | Minimum value |
| MAX | Maximum value |
| FIRST | First value |
| LAST | Last value |
| STDDEV | Standard deviation |
| VARIANCE | Variance |
| ARRAY_AGG | Collects group values into an array |

- For COUNT, you can select `* (all)` as the column to aggregate.
- FIRST and LAST require you to specify a sort column and direction (ascending/descending), and return the first or last value after sorting.

Selecting ARRAY_AGG adds the following settings:

| Item | Required | Description |
| --- | --- | --- |
| Array element type | O | Single column or JSON object |
| Column to collect | O* | Required when the array element type is a single column. The column to include in the array |
| JSON object field definition | O* | Required when the array element type is a JSON object. A combination of key names and columns |
| Array element sort | X | Sort column and direction. If not specified, no sorting is applied |
| Output format | O | STRUCT_ARRAY, JSON_STRING_ARRAY, JSON_STRING |
| Deduplication (DISTINCT) | X | Ignored if sorting is configured |

<a id="pipeline.node.transform.rank"></a>
##### Rank top N { #pipeline.node.transform.rank }

Sorts all data by the specified criteria and retains only the top N rows.

| Item | Required | Description |
| --- | --- | --- |
| Sort criteria | O | The column to sort by and the direction (DESC / ASC) |
| Top N rows | O | The number of rows to include in the result after sorting |
| Result rank column name | O | The name of the rank column (values 1 through N) added to the result |

- You can specify multiple sort criteria using the **Add Sort Criteria** button. They are applied in order (primary sort, secondary sort, and so on), and you can drag to reorder them.
- When querying, sorting by the rank column preserves the original order.

<a id="pipeline.node.transform.label.encode"></a>
##### Label encoding (Label Encode FIT / Label Encode Apply) { #pipeline.node.transform.label.encode }

This feature converts categorical data into numeric (integer) values. It consists of two stages: **FIT (training)** and **APPLY (application)**. You must create a FIT node first and then reference it in the APPLY node.

FIT node settings:

| Item | Required | Description |
| --- | --- | --- |
| Select columns to encode | O | The columns to apply label encoding to (at least one) |
| Start index | X | The starting value for encoding (default: 0) |

APPLY node settings:

| Item | Required | Description |
| --- | --- | --- |
| Select mapping table node | O | Select the FIT node |
| Overwrite source column | X | When checked, replaces the source column with the encoded value (default: unchecked) |
| Result column suffix | X | The suffix appended to new column names when the source column is retained (default: _encoded) |

!!! tip "Note"
    Values not present in the mapping table (OOV, Out-of-Vocabulary) are encoded as `-1`.

!!! danger "Caution"
    Re-running FIT may change the encoding numbers depending on changes to the data. Be careful about compatibility with existing models.

<a id="pipeline.node.transform.classify"></a>
##### Classify { #pipeline.node.transform.classify }

Classifies the values of a text column into categories using an LLM. The category list used as the classification criteria must be prepared as a separate data source node.

When the pipeline runs, it groups input data rows by the batch size and calls the LLM once per batch. Each call's prompt includes the category list and the rows of that batch. When the LLM responds with the appropriate category ID for each row, the result is stored in the classification result column.

**LLM Settings**

| Item | Required | Description |
| --- | --- | --- |
| LLM provider | O | Claude |
| Model name | O | claude-haiku-4-5, claude-sonnet-4-6 |
| API key | O | The API key for the LLM. Stored only for this pipeline and not displayed again after saving |
| Batch size | X | The number of rows per LLM call (1–30) |
| Max retries | X | The number of retries on call failure (0–10) |
| Timeout (seconds) | X | The call wait time (1–600) |
| Incremental classification | X | Classifies only changed rows using the LLM and reuses previous run results for unchanged rows |

**Input Columns**

| Item | Required | Description |
| --- | --- | --- |
| Input column | O | The columns to pass to the LLM. Multiple columns can be selected |
| Column join delimiter | X | The delimiter used when concatenating multiple input column values |
| User prompt template | X | A template for composing input values per row. The {{ column name }} pattern is replaced with the corresponding column value. When set, this takes priority over the column join delimiter. Only columns selected in the input column field can be referenced |

**Axis (Category Table)**

Register the category list used as classification criteria as an axis. You can add up to five axes.

| Item | Required | Description |
| --- | --- | --- |
| Category node | O | The data source node that contains the category list |
| Category ID column | O | The column that identifies the category |
| Category name column | O | The column that contains category names |
| Category metadata column | X | Additional columns such as category descriptions and classification criteria. Passed along in the prompt to improve classification accuracy |
| Classification result column (array) | O | The name of the array column to store classification results |
| Representative column | X | The name of the column to store the most representative classification ID. If left empty, the column is not created |

**Output**

| Item | Required | Description |
| --- | --- | --- |
| Output type | O | SINGLE_LABEL, MULTI_LABEL |
| Classification rationale column | X | The name of the column to store the classification rationale provided by the LLM. If left empty, the rationale is not requested, reducing token costs |
| Content rationale column | X | The name of the column to store a short description generated from the input content |
| Max content rationale length | X | The maximum number of characters for the content rationale column (1–200) |

**System prompt** (optional): Instructs the LLM on its role and classification criteria. If not set, the default prompt is used.

!!! danger "Caution"
    As the number of rows to process increases, the number of LLM calls and token usage increase proportionally, raising costs. For large datasets, validate on a small scale first to verify cost and quality before applying to the full dataset.

<a id="pipeline.node.join"></a>
#### Join node (JOIN) { #pipeline.node.join }

Combines two data streams.

| Item | Required | Description |
| --- | --- | --- |
| Node name | O | The name of the join result node (up to 30 characters) |
| Join type | O | Select from Inner Join / Left Join / Right Join / Full Outer Join / Semi Join (EXISTS) / Anti Join (NOT EXISTS) |
| Join tables | O | Select left/right table nodes |
| Join condition | O | Left field = right field mapping (multiple conditions are supported) |
| Column selection | X | Select columns to include in the result; a prefix can be set |

!!! tip "Note"
    If both tables have a column with the same name, you can set a **prefix** on the right table to prevent column name conflicts.

<a id="pipeline.node.union"></a>
#### Union node (UNION) { #pipeline.node.union }

Vertically merges two data streams.

| Value | Description |
| --- | --- |
| Full Merge | Includes all columns from both sides. Missing values are NULL |
| Intersect Merge | Includes only columns common to both sides |
| Left-First Merge | Based on the left table's schema. Columns missing from the right side are NULL |

<a id="pipeline.node.dataset"></a>
#### Dataset node (DATASET) { #pipeline.node.dataset }

The final output node of a pipeline. Saves processed data as a table.

| Item | Required | Description |
| --- | --- | --- |
| Dataset name | O | Used as the table name (up to 30 characters). Only lowercase English letters, numbers, and underscores (_) are allowed; must not start with a number |
| Description | X | Dataset description (up to 500 characters) |
| Column selection | O | Select the columns to include in the dataset (at least one) |

The dataset name is used as both the data source name and the table name, so it cannot duplicate an existing data source name or table name. Duplicate checks are performed both when the pipeline is saved and when it is run.

!!! danger "Caution"
    A dataset that has been run cannot be modified. Delete it and recreate it (the data source is retained).

<a id="pipeline.schedule"></a>

### Schedule Settings { #pipeline.schedule }

Click the schedule icon in the right side panel to configure the batch schedule.

| Interval Type | Configuration | Example |
| --- | --- | --- |
| Every minute | Interval (5/10/15/20/30 minutes) | Execute every 10 minutes |
| Every hour | Interval (1/2/3/4/6/8/12 hours), minutes | Execute every 2 hours at 30 minutes |
| Every day | Hour, minute | Execute every day at 9:30 AM |
| Every week | Day of week (multiple selection), hour, minute | Execute every Monday, Wednesday, and Friday at 9:00 AM |
| Every month | Day, hour, minute | Execute every 1st of the month at 9:30 AM |

Date range settings:

| Item | Required | Description |
| --- | --- | --- |
| Start date setting | X | When checked, the schedule starts from the configured date. If not set, the schedule starts from the time it is saved. |
| End date setting | X | When checked, the schedule does not run after the configured date. |

After the schedule is saved and runs for the first time, it runs according to the configured interval.

<a id="pipeline.resource"></a>


### Computing Resource Settings { #pipeline.resource }

Click the computing resource icon in the right side panel to configure the computing resources to use when running the pipeline.

| Type | Driver | Executor |
| --- | --- | --- |
| Small (Default) | 1 CPU, 1 GB memory | 1 CPU, 1 GB memory × 1 |
| Medium | 1 CPU, 2 GB memory | 1 CPU, 2 GB memory × 2 |
| Large | 2 CPU, 4 GB memory | 2 CPU, 4 GB memory × 4 |
| XLarge | 4 CPU, 8 GB memory | 4 CPU, 8 GB memory × 8 |

!!! danger "Caution"
    If you change the resource settings, the changes are applied after a rebuild. You cannot change the resources before the pipeline is created.

### Run a Pipeline { #pipeline.run }

On the first run or the first run after a configuration change, the build and execution proceed together.

1. Click **Run** on the tab bar.
2. Click **Run** in the confirmation modal.

The pipeline runs with the last saved configuration. If you have unsaved changes, a confirmation modal appears first to notify you that the pipeline will run with the configuration before the changes were saved. To include the changes in the run, save them first.
A pipeline that has already been built runs immediately without a build process.
Clicking **Stop** during execution transitions the pipeline to the Terminated state.
When the build is complete, pipelines with a schedule configured are automatically activated.

<a id="pipeline.run.history"></a>
#### View Run History { #pipeline.run.history }

Click **Run History** on the tab bar to view the run ID, start/end date and time, duration, run status, and run details per node.

<a id="pipeline.activation"></a>

### Activate/Deactivate { #pipeline.activation }

- **Activate**: Select one pipeline from the list and click **Activate Pipeline** in the More (⋯) menu. The pipeline runs automatically according to the configured schedule.
- **Deactivate**: Select one pipeline from the list and click **Deactivate Pipeline** in the More (⋯) menu. The pipeline does not run automatically even if a schedule is configured.

<a id="pipeline.delete"></a>

### Delete a Pipeline { #pipeline.delete }

1. Select the pipeline to delete from the list (multiple selections are allowed).
2. Click **Delete**.
3. Click **Delete** in the confirmation modal.

<a id="query"></a>

## Analytics - Query { #query }

Console path: **Machine Learning > NHN Cloud Foundry > Analytics** tab > **Query** tab

Query and analyze data from data sources using SQL.

<a id="query.run"></a>
### Run a Query { #query.run }

1. Select a **Data Source**.
2. Load a saved query from **Select Query**. Saved queries for the selected data source appear in the list. You can also write a query directly without loading one.
3. Enter an SQL statement in the query input field.
4. Select a **Row Limit** (10, 100, 1,000, 10,000, or 100,000; default is 1,000).
5. Click **Run Query**.

- The execution results are displayed in a data grid format. The column structure is dynamically generated based on the query results, and pagination is supported.
- When you select a data source, a schema panel is displayed to the right of the query input field, where you can view field names and data types. You can search by field name.
- Press **Ctrl+Enter** (or **⌘+Enter** on macOS) in the query input field to run the query.
- If the query fails, the reason for the failure is displayed in the results area.
- Click **Reset** to clear the query that you entered.
- Click **Download Query Results** to save the results as a CSV file (file name: `query name_date_time.csv`).
- In the FROM clause, use the table name exactly as displayed in the data source list (`SELECT * FROM {table name}`).
- Only a single SELECT statement can be executed. All other statements are rejected.

<a id="query.save"></a>
### Edit a Saved Query { #query.save }

After modifying a query loaded via **Select Query**, click **Save Query** to save the changes. The button is disabled if no query is loaded or if the content has not been changed.

<a id="query.list"></a>
### Query List { #query.list }

Each query execution is automatically recorded in the execution history. You can also create queries that you reuse directly in the query list. Click **Query List** to open a modal that allows you to manage saved queries.

| Column | Description |
| --- | --- |
| Query Name | Name specified when saving |
| Query Statement | Saved SQL |
| Saved At | Date and time the query was last saved |
| Details | View the full query content |

- You can search by query name.
- Click **Add** to register a new query. Select a query and click **Edit** or **Delete** to edit or delete it. Deleted queries cannot be recovered.

The following fields are required when adding or editing a query:

| Field | Required | Description |
| --- | --- | --- |
| Query Name | O | Name used to identify the query |
| Data Source | O | Data source on which to run the query |
| Statement | O | SQL to execute |

<a id="query.history"></a>
### Query Execution History { #query.history }

This screen displays the history of all queries that you have run. Search by **Time** range and **Query Content**, and click **Reset** to clear the search conditions.

| Column | Description |
| --- | --- |
| Query Name | Name of the executed query |
| Query Execution Date | Date and time the query was executed |
| Data Source Name | Data source on which the query was executed |
| Query/Result | Executed SQL and results |

Click an item in the list to view the query details. Click **Use Query** to load the query into the execution screen.

<a id="chart"></a>

## Analysis - Chart { #chart }

Console path: **Machine Learning > NHN Cloud Foundry > Analysis** tab > **Chart** tab

<a id="chart.list"></a>

### Chart List { #chart.list }

- You can filter charts by name using the search feature at the top.

<a id="chart.create"></a>

### Create Chart { #chart.create }

Click the **Create Chart** button to navigate to the chart editor. Configure the following settings in the editor.

<a id="chart.create.basic"></a>

#### Basic Settings { #chart.create.basic }

| Item | Required | Description |
| --- | --- | --- |
| Chart name | O | Name to identify the chart (up to 30 characters) |
| Chart type | O | Defines the chart's purpose (currently, only the default type is available) |
| Chart visualization type | O | Line Chart, Bar Chart, Pie Chart, Scatter Chart |

<a id="chart.create.datasource"></a>

#### Data Source Settings { #chart.create.datasource }

| Item | Required | Description |
| --- | --- | --- |
| Data source type | O | FILE, DATASET, RECOMMENDATION SINK |
| Data source name | O | Select the data source to use |

<a id="chart.create.query"></a>

#### Query Settings { #chart.create.query }

| Item | Required | Description |
| --- | --- | --- |
| X-axis | O* | Field to use on the horizontal axis. Not displayed for Pie charts |
| Aggregation interval | O* | 5 minutes, 15 minutes, 30 minutes, 45 minutes, 1 hour, 2 hours, 6 hours, 12 hours, 1 day, 3 days, 7 days. Displayed only for Line and Bar charts |
| Reference time | O | Specifies the time range for the data |
| Column | O | Select the column to aggregate and the aggregation function. At least one is required |
| Group key | O* | Column to use for grouping values. At least one is required for Pie charts; optional for other chart types |
| Filter | X | Sets the data filtering conditions |
| Sort | X | Sort column and direction. Displayed only for Pie and Bar charts |
| Row limit | O | Maximum number of rows to retrieve |

The required settings for each chart visualization type are as follows:

| Chart Visualization Type | Required Settings |
| --- | --- |
| Line, Bar | X-axis (time axis), aggregation interval, at least one column |
| Pie | At least one column, at least one group key |
| Scatter | At least one column |

When sorting is specified for a Bar chart, the chart displays the top N categories instead of a time axis.

<a id="chart.create.preview"></a>

#### Chart Preview { #chart.create.preview }

After completing the configuration, click the **UPDATE CHART** button to preview the chart.

- Verify that the chart is displayed correctly.
- You can check the data in the table view at the bottom, and toggle the table view between shown and hidden.

<a id="chart.create.save"></a>

#### Save Chart { #chart.create.save }

- After confirming the preview, click the **Create** button in the header.
- The Create button becomes active when input validation is complete.

<a id="chart.edit"></a>

### Edit Chart { #chart.edit }

Click a chart in the chart list to open the edit screen.

- You can modify the basic settings, data source settings, and query settings.
- Preview the changes with **UPDATE CHART**, then click **Save** to save the changes.

<a id="chart.delete"></a>

### Delete Chart { #chart.delete }

1. Select the chart to delete from the chart list using the checkbox.
2. Click the **Delete** button.
3. Click **Delete** in the confirmation modal.

!!! danger "Caution"
    Deleting a chart that is in use on a dashboard also removes it from that dashboard.

<a id="dashboard"></a>

## Analysis - Dashboard { #dashboard }

Console path: **Machine Learning > NHN Cloud Foundry > Analysis** tab > **Dashboard** tab

<a id="dashboard.list"></a>
### Dashboard List { #dashboard.list }

- You can filter dashboard names using the search function.
- You can adjust the number of items displayed per page.

<a id="dashboard.create"></a>
### Create a Dashboard { #dashboard.create }

Click the **Create Dashboard** button to go to the dashboard editor screen.

| Item | Required | Description |
| --- | --- | --- |
| Dashboard name | O | Name to identify the dashboard |
| Dashboard description | X | Description of the dashboard |

The edit panel consists of two tabs.

| Tab | Description |
| --- | --- |
| CHARTS | List of charts that can be added to the dashboard. Searchable by chart name |
| LAYOUT ELEMENTS | Layout elements to add to the canvas |

<a id="dashboard.create.chart"></a>
#### Add a Chart { #dashboard.create.chart }

1. Click or drag a chart card to add it.
2. Place the chart on the dashboard canvas.

<a id="dashboard.create.tabgroup"></a>
#### Add a Tab Group { #dashboard.create.tabgroup }

You can group multiple charts into tabs and switch between them in one place. Click **Tab** in the **LAYOUT ELEMENTS** tab of the edit panel to add a tab group to the canvas.

- Use **Add Tab** in the tab group to add more tabs. When there are two or more tabs, a delete icon appears on each tab. If only one tab remains, it cannot be deleted.
- Double-click a tab to change its name.
- Drag a chart from the edit panel and drop it onto a tab area to place it in that tab. For charts already on the canvas, drag the handle on the left side of the chart title to move the chart into a tab. You can return a chart from a tab back to the canvas using the same method.
- To delete a tab group from the dashboard, click the trash icon in the top right corner, just as you would with a chart.

<a id="dashboard.create.layout"></a>
#### Adjust the Layout { #dashboard.create.layout }

Drag a chart on the canvas to reposition it, and drag its corner to resize it. Click the trash icon in the top right corner of a chart to remove it from the dashboard.

<a id="dashboard.create.save"></a>
#### Save the Dashboard { #dashboard.create.save }

After completing the configuration, click the **Save** button in the header.

<a id="dashboard.view"></a>
### View a Dashboard { #dashboard.view }

Click a dashboard in the dashboard list to go to its detail screen.

<a id="dashboard.edit"></a>
### Edit a Dashboard { #dashboard.edit }

On the dashboard detail screen, click the **Edit Mode** toggle to switch to edit mode.

In edit mode, you can perform the following actions:

- Add, delete, and reposition charts
- Change the dashboard name

To modify a chart placed on the dashboard, turn off edit mode. When edit mode is off, a more options menu appears on each chart. Choose **Edit Chart** to go to the chart editing page. Charts inside a tab group can be accessed the same way. After editing and saving a chart, the changes are reflected in the dashboard.

<a id="dashboard.delete"></a>
### Delete a Dashboard { #dashboard.delete }

1. In the dashboard list, select the dashboard to delete.
2. Click the **Delete** button.
3. Click **Confirm** in the confirmation modal.

<a id="app"></a>

## App { #app }

Console path: **Machine Learning > NHN Cloud Foundry > App** tab

You can create and manage recommendation system serving pipelines that use AI models.

<a id="app.list"></a>
### App List { #app.list }

| Column | Description |
| --- | --- |
| App Type | Type of the app |
| App ID | Unique identifier for the app |
| App Name | Name used to identify the app |
| App Description | Description of the app |
| Status | Status of the app |
| Created On | Date and time the app was created |

<a id="app.list.status"></a>
#### App Status { #app.list.status }

Hover over an app status badge to view a detailed description in a tooltip.

| Value | Description |
| --- | --- |
| Initializing | App initialization in progress |
| Training | AI model training in progress |
| Deploying | App deployment in progress |
| Activating | App activation in progress |
| Active | App is active |
| Deleting | App deletion in progress |
| Failed | An error occurred while processing the app |
| Unknown | Status information unavailable |

- After the app is created, training and deployment proceed automatically.
- If an error occurs at any stage, the status changes to Failed.

!!! tip "Note"
    The training and deployment that begin immediately after app creation are part of the app preparation process. At this point, the recommendation model has not yet been trained. Calling the recommendation API returns a response, but the response does not reflect the results of a trained model.
    The first training run is executed at the time specified in the batch schedule (status: Training → Deploying → Activating → Active). Valid recommendation results are available after the trained model is deployed.

<a id="app.create"></a>
### Create App { #app.create }

Click the **Create App** button to go to the app creation screen. App creation consists of three steps.

| Step | Description |
| --- | --- |
| Basic Settings | Select app name, description, and type |
| Advanced Settings | Select model, serving resources, batch schedule, data connections, and additional settings |
| Final Review | Confirm inputs and create the app |

<a id="app.create.basic"></a>
#### Basic Settings { #app.create.basic }

| Item | Required | Description |
| --- | --- | --- |
| App Name | O | Name used to identify the app (up to 255 characters). Supports Korean, Japanese, English, numbers, spaces, hyphens (-), and underscores (_) |
| App Description | O | Description of the app |
| App Type | O | Select **Recommendation System** |

<a id="app.create.detail"></a>
#### Advanced Settings { #app.create.detail }

Use the **Add Model** button to add model cards. You can configure multiple models in a single app, and each model card has the sections described below.

<a id="app.create.detail.model"></a>
##### Basic Model Settings { #app.create.detail.model }

| Item | Required | Description |
| --- | --- | --- |
| Model Name | O | Select the recommendation model to use |
| Longtail Mode | X | When enabled, includes less popular items in recommendations |

The following models are available:

| Name | Description |
| --- | --- |
| Cold User | Recommends items based on item attributes to users with limited activity history, such as new users |
| Warm User (Transformer) | Recommends the next item based on the sequence of items a user recently interacted with |

Even the same model can be added as a separate entry if the Longtail Mode setting differs.

<a id="app.create.detail.resources"></a>
##### Serving Resource Settings { #app.create.detail.resources }

Specifies the resources to allocate to the serving container for each model. If left empty, default values are applied.

| Item | Required | Description |
| --- | --- | --- |
| CPU request / CPU limit | X | In cores. Default: 2 / 4 |
| Memory request / Memory limit | X | In Gi. Default: 4 / 4 |

`request` cannot exceed `limit`.

<a id="app.create.detail.schedule"></a>
##### Batch Schedule Settings { #app.create.detail.schedule }

Sets the execution frequency of the batch that retrains the model. This is enabled by default and can be turned off using a toggle.
The first training run of the model is also executed at the time specified in this schedule. Recommendation responses before the first training is complete do not reflect the results of a trained model.

| Frequency | Settings |
| --- | --- |
| Daily | Hour, Minute |
| Weekly | Day of the week, Hour, Minute |
| Hourly | Interval (hours), Minute |

The specified time is applied based on the time zone of the browser you are using.

<a id="app.create.detail.connection"></a>
##### Data Connection Settings { #app.create.detail.connection }

Connect the data sources required based on the selected model.

General recommendation model:

| Item | Required | Description |
| --- | --- | --- |
| User Data Source | O | Data source for user information |
| User ID Column | O | Column used to identify users |
| User Feature Columns | X | Additional user attribute columns (multiple selections allowed) |
| Item Data Source | O | Data source for item information |
| Item ID Column | O | Column used to identify items |
| Item Feature Columns | X | Additional item attribute columns (multiple selections allowed) |
| History Data Source | O | Data source for user-item interaction history |
| History User ID Column | O | Column used to identify users in the history data |
| History Item ID Column | O | Column used to identify items in the history data |
| Timestamp Column | X | Column for interaction timestamps |
| History Feature Columns | X | Additional interaction attribute columns |

Tag embedding model:

| Item | Required | Description |
| --- | --- | --- |
| Tag Data Source | O | Data source for tag information |
| Attribute Column | O | Column for tag values |
| Item ID Column | O | Column used to identify items |
| Timestamp Column | X | Timestamp column |

<a id="app.create.detail.extra"></a>
##### Additional Settings (Skills) { #app.create.detail.extra }

This is an optional setting for connecting skill and category data used to configure recommendation reasons. Use the **Add Field** button to add entries.

| Item | Description |
| --- | --- |
| Skill Data Source | Skill information table. Specify the skill ID column and skill column |
| Default Skill Category / Common Skill Category | Skill classification table. Specify the skill ID column and skill label column |
| User Group Data Source | Unit for grouping recommendation target users (e.g., department, grade, interest group). Specify the user group ID column and skill column |
| User Interest Skill Data Source | Per-user interest skill table. When interest skills are not passed in the recommendation API, this table is queried to generate interest-based recommendation reasons |
| User Attribute Mapping | Specifies the key names for passing each item via userAttributes in the recommendation API |
| Recommendation Reason Template Data Source | Template table for recommendation reason text. If not selected, no reason is included in the recommendation results |
| Cold Start Data Source | Only user IDs listed in this table are classified as cold starters. Both the data source and user ID column must be selected |

<a id="app.create.review"></a>
#### Final Review { #app.create.review }

| Review Item | Description |
| --- | --- |
| Basic Settings | App name, description, and type |
| Model Settings | Selected model, serving resources, batch schedule, and data connection information |
| Additional Settings | Additional settings such as skill tables |

Click the **Save** button to create the app. On success, a completion dialog is displayed and you are redirected to the list. On failure, an error message is displayed.

<a id="app.delete"></a>
### Delete App { #app.delete }

1. Select the checkbox for the app you want to delete.
2. Click the **Delete** button.
3. Click **Confirm** in the confirmation dialog.

!!! danger "Caution"
    Deleted apps cannot be recovered. The associated serving pipeline is also deleted.

<a id="app.detail"></a>
### App Details { #app.detail }

Click an app in the app list to go to the details screen. The details screen consists of two tabs: **Call Recommendation API** and **App Info**.

<a id="app.detail.recommend"></a>
#### Call Recommendation API { #app.detail.recommend }

You can call the recommendation API directly by entering recommendation request parameters and viewing the results. The screen consists of three areas: an input form, a request preview, and recommendation results.

Input form:

| Item | Description |
| --- | --- |
| Recommendation App ID | App ID to call. Automatically populated with the current app |
| User ID | Select the target user for recommendations |
| Recommendation Mode | Select from Normal Flow (history-based) or Cold Start (attribute-based) |
| Max Recommendations | Maximum number of items to include in the response (1–100, default: 10) |
| Longtail Mode | Improves recommendation diversity by including less popular items. Not available in Cold Start mode |
| context | Add contextual information for the recommendation request (e.g., current or recently viewed items) as individual fields |
| userAttributes | Add user attribute information (e.g., group, age, interests) as individual fields |
| options | Add recommendation request options as individual fields |

- **Request Preview**: Displays the actual API request JSON built from the input values. You can click the **Copy** button to copy it for use in API integration development.
- **Recommendation Results**: Click the **Request Recommendation** button to view the rank, item key, score, and recommendation reason. The total number of results and response time are also displayed.

!!! tip "Note"
    Recommendation API calls are only available when the app is in the Active state.

<a id="app.detail.info"></a>
#### App Info { #app.detail.info }

You can view the app ID, app name, status, app type, description, creation date, modification date, version, and input and output data sources.

- Input data sources: Data sources used for model training. For recommendation apps, these are displayed separately by model.
- Output data sources: Data sources where recommendation results are stored.