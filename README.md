# CSVServer

Simple node-based CSV adapter for Grafana Simple JSON data source

This is a simple node application to read table or timeseries formatted CSV files as JSON, sutable for Granfana's SimpleJSON data source

This is intended for testing and demonstration purposes only - not recommended for production environments where efficiency and robustness are needed.

## Installation/setup

From a command window, execute the following commands:

```javascript
git clone https://github.com/michaeldmoore/CSVServer.git

npm install

node CSVServer.js
```

Using these default settings, CSVServer serves CSV files from the default data folder (csv)

The data folder can be overriden using a single parameter on the command line, line this:

```javascript
node CSVServer.js my-data-directory
```

Alternatively, the default data folder can be specified with a command line option or - on a query-by-query basis, as parameters in Grafana queries 'Additional JSON data' fields.  See below.

Note:  In this initial release, the date-range filters used by Grafana was disabled by default, so that ALL the data in the CSV file was returend to the client, irrespective of the current Grafana display date/time range.  This, while possibly being inefficient in a real-world/production environment where data sets could become very large, could be useful when debugging queries.  The current release turns this option ON by default.  See below for command line and per-query options allowing this to be set or reset dynamically.

## Sample table formatted CSV file

```javascript
datetime,sensor,region,value
2017-09-05 18:21:07.000,AAA,West,1.234
2017-09-05 21:18:42.000,AAA,West,1.234
2017-09-05 21:21:32.000,AAA,West,1.234
2018-04-17 10:39:12.000,AAA,West,1.234
2018-04-17 10:39:12.000,DDD,South,0.51
2017-09-05 21:18:42.000,EEE,West,0.9
2018-01-01 01:01:01.000,EEE,West,0.5
2018-04-17 10:39:12.000,EEE,West,0.3
2017-09-05 21:21:32.000,FFF,Central,0.160411622524645
2018-04-17 10:39:12.000,FFF,Central,0.838865293039815
```

Each row should have at least one numeric value column (though this can ba called anything).  Each row in the csv file will return a row in the JSON datasource, with the other columns available as named fields.  Generally, there will be a date/time field too, though this is not strictly necessary.

## Sample single-valued time-series formatted CSV file

```javascript
date,temperature
2018-12-18 00:00:00.000,51.5
2018-12-18 01:00:00.000,56.2
2018-12-18 02:00:00.000,58.3
2018-12-18 03:00:00.000,62.4
2018-12-18 04:00:00.000,66.5
2018-12-18 05:00:00.000,70.6
2018-12-18 06:00:00.000,65.7
2018-12-18 07:00:00.000,45.8
```

## Sample multi-valued time-series formatted CSV file

```javascript
val1,val2,date,val3
90,73,2020-04-03 10:00,37
80,84,2020-04-03 10:30,46
70,80,2020-04-03 11:00,55
70,80,2020-04-03 11:30,55
60,65,2020-04-03 12:00,44
85,46,2020-04-03 12:30,54
74,67,2020-04-03 13:00,63
68,58,2020-04-03 13:30,85
71,79,2020-04-03 14:00,65
71,79,2020-04-03 14:30,65
```

Each row should have a date field (by default, named 'date' though this can be overriden - see below), plus one or more uniquely named value fields.

CSVServer runs as a HTTP (REST) server, on port 4000 (this can be changed by editting CSVServer.js).  Also, note that the current version is set to run on the localhost (ie, on the same server as Grafana itself).  Hosting CSVServer on a different server will most likely require a CORS header to be added to the source, and any firewalls configured to support it).  Adding secure HTTPS support would require a similar change.  No plans exist to build this support in - at least - not yet.

## Setting up Grafana SimpleJSON datasource to use CSVServer

The SimpleJSON datasource plugin needs to be installed first.  If not already installed, stop Grafana server and run the following command line (windows):

```javascript
grafana-cli plugins install grafana-simple-json-datasource
```

Newer versions of Grafana provide an alternative, presumably enhanced, datasource named simpll JSON.  This appears to be completely backwards compatible with SimpleJSON.  Install the JSON datasource instead using the alternate command line (windows)

```javascript
grafana-cli plugins install simpod-json-datasource
```

Then restart Grafana server (See <https://grafana.com/plugins/grafana-simple-json-datasource/installation> for details, if needed)

Open Grafana as a client in a web browser and log in as usual (this may require admin rights within Grafana).  Select new data source, provide a data source name (CSVServer suggested) and define the type as SimpleJSON.
![showcase](https://user-images.githubusercontent.com/3724718/50186644-141f3f00-02e1-11e9-812f-94be59ea3d92.png)

Now define the URL to point to the node server, and click on 'Save & Test'
![showcase](https://user-images.githubusercontent.com/3724718/50186753-75471280-02e1-11e9-8382-68606c0a373f.png)

The SimpleJSON configuration screen has a number of other options which might be useful under certain circumstances - generally the defaults should work.

Once in operation, the CSV files can be updated dynamically - each data request from Grafana will cause the files to be re-parsed, so you should be able to see the effects immediately.

## Server options/overrides

By default, CSVServer serves CSV files from a subfolder ('csv').  This, as mentioned above can be overriden using a default parameter on the command line used to start CSVServer (see above)

Additionally, this and several other options can be specified on the start-up command line to change the default behaviour of CSVServer.

Starting CSVServer with a command line option --help, shows the list of supported command line options.
![image](https://user-images.githubusercontent.com/3724718/78253512-67280000-74ec-11ea-8900-332e7e49ebc3.png)

### --folder

Override the CSV file folder - this is the default option, so specifying a folder name at the end of the command line has the same effect as specifying it via this command line option.

### --datecols

By default, CSVServer treats columns named 'date' as a date/time field.  All other columns are considered to e numerical values.  The --datecols parameter replaces this default name with an alternative - or a list of alternatives - each seperated by a comma (,) character.  Fields matching any of these names are treated as date/time values.

### --dateformat

By default, CSVServer expects the first field of CSV files involved in timeseries quesries to be formatted in ISO standard format - that means, something like 'YYYY-MM-DD HH:mm:ss' (plus the optional mS and timezone extensions).  Non ISO-formatted date/time fields, such as commonly used in the US (MM-DD-YYYY) and europe (DD-MM-YYYY) cannot be parsed correctly using the default setting.  This option allows the datetime formatting string to be set for all queries.  Note, the formatting characters are case sensative.  See <https://devhints.io/moment> for a cheat-sheet of the formatting codes recognized by moment.js, the library used for this purpose.

### --nodatetimefilter

By default, CSVServer filters out records where the datetime field falls outside the query date/time range.  Sometimes, it can be helpful to turn this filtering off, leaving it up to the Grafana display panels to decide which records to display.  Setting this option on the command line turns this internal filtering off, so every row in the CSV file will be returned to Garafana regardless of the current display date/time range.

## Per-query overrides (Grafana query 'Additional JSON data')

In addition to the start-up command lines options described above, CSVServer can process per-query option overrides through the Additional JSON data field in the query editor.
Each of the options mentioned in the previous section can be applied as per-query Additional JSON Data elements apart from the folder override.  Overriding the CSV data folder could create security concerns.

See the sample query definition below, reading a set of 3 simple time series - each having a different date/time format (ISO, UK and US).
![image](https://user-images.githubusercontent.com/3724718/78248918-663fa000-74e5-11ea-8659-16dc5bba05b5.png)
