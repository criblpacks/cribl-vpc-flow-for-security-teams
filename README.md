# AWS VPC Flow Pack for security teams
----
This pack allows you to enrich, aggregate, selectively suppress VPC Flow Log events. Optionally, the pack makes the events Splunk CIM compliant. The pack has a series of Function Groups that can be enabled or disabled to achieve your data reduction, enrichment and normalization requirements. 
 
Benefits of using this pack:
- significantly reduce data volume going to the analytics tools by using event filtering, aggregation and suppression
- enrich data with contextual information like host names to simplify and accelerate security data analysis
- enrich data with GeoIP information at data ingest time, not at search time when the GeoIP mapping may become outdated
- make data Splunk CIM compliant for Splunk ES and InfoSec apps

## Requirements
---
Before you begin ensure that you have met the following requirements:

- Create a Route with a filter for your AWS VPC Flow events.
- Select the *AWS VPC Flow for Security Teams* pack as the pipeline.
- If you plan to aggregate events and enrich events with frequently changing host information, provision a Redis instance and specify Redis URL in the Redis functions of the pack.

To use GeoIP lookups, upload .mmdb database from MaxMind. If you are not signed up for MaxMind database yet, [start here](https://www.maxmind.com/en/geolite2/signup), download GeoLite2 City GZIP file, unzip it and add GeoLite2-City.mmdb to lookups under Knowledge menu.

## Warning about index name
---
If sending VPC Flow Logs to Splunk, please note that this pack does not add or change index name. Consider adding index name to avoid data going to the default Splunk or Splunk Cloud index.

## VPC Flow Logs version
---
The pack expects VPC Flow Log version 2. Version 2 is the default version. If you use a different version you can modify the list of the fields to match your version in Function #4 of the VPC-Flow-logs-enrichment pipeline. See [this document](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html#flow-logs-fields) for more information about the VPC Flow Logs versions. 

## Function Groups and how to use them in the pack
---
Enable or disable groups based on desired outcome. For example, if you don't use Splunk, turn off the Group called *Group 6: Make data Splunk CIM compliant*. Do not disable Groups that are marked as *REQUIRED*.
 
## Output formats
---
Chose between JSON and CSV output format. 
- CSV will produce significantly lower data volume but will require you to define how your security analytics tools interpret the CSV data.
- JSON will likely allow your analytics tools to read key/value pairs with no extra steps required but that will result in higher data volume.
  
## Redis
---
This pack uses Redis (in-memory database) to aggregate and suppress similar events at scale.
The pack also uses Redis to optionally enrich data with asset information.
More information on using Redis with LogStream can be found here: https://docs.cribl.io/docs/redis-function 


## Expected format for asset information
---
Lookup table *assets.csv* under *Knowledge* tab on the pack shows the expected fields/column for the asset information.
If you use Redis for the asset information, use the same field names as in the *assets.csv* lookup table.

## Note on aggregation of VPC Flow Logs with Aggregations function
---
On higher VPC Flow Log volumes aggregation with Logstream Aggregations function may not be effective for the following reasons:
- Aggregation will be done per Worker Process, not across all your LogStream workers.
- High volume flow logs will require significant memory on Worker Nodes to achieve better aggregation results.
- VPC Flow Logs are already aggregated in AWS. To take advantage of further aggregation, the in-memory aggregation window should be meaningfully larger than the aggregation window configured for the logs in AWS. The pack's default aggregation window is 30 minutes.

## Aggregation using Redis
---
When using Redis, it keeps the state of data aggregation accross Workers. The pack uses the followig logic for event aggregation:
- ACCEPT and REJECT events are aggregated differently.
- For ACCEPT events aggregation is done over 60 minutes or when bytes transferred between source and destimation exceed 1,000,000 whichever occures first. Not all the source-to-destination communication events will exceed 1,000,000 bytes; for this reason the first communication source-to-destination at the begining of aggregtion window will be passed to your destination to ensure no source-to-destination pairs are missing in your analytics tool.
- For REJECT events aggregation is done over 60 minutes regardless of bytes transferred. Bytes transferred is not passed to the destination for those dropped events.


## Splunk Common Information Model (CIM) normalization
---
If you selected JSON output format, you can also enable the Splunk CIM normalization Group that changes field names to map to Splunk CIM. 
NOTE: You still need to add tags in Splunk. VPC Flow Logs need two tags: tag=*network* and tag=*communicate*. More information on the applicable Splunk CIM Data Model: https://docs.splunk.com/Documentation/CIM/latest/User/NetworkTraffic 

## Release Notes
---
### Version 0.7.3 - 2021-11-17
More detailed instructions added to this README.
Warning about index name added to this README.

### Version 0.7.2 - 2021-11-08
Instructions for adding MaxMind GeoIP lookup are added.

### Version 0.7.1 - 2021-11-03
Redis based aggregation and suppression options added
Function groups including Redis are turned off by default

### Version 0.5.0 - 2021-09-15
Initial release. Enrichment of AWS VPC Flow events with EC2 instance details, Geo IP information from MaxMind database, event aggregation and Splunk CIM compliance. 

## Contributing to the Pack
---
Discuss this pack on our Community Slack channel [#packs](https://cribl-community.slack.com/archives/C021UP7ETM3).

## Contact
---
The author of this pack is Igor Gifrin and can be contacted at <igifrin@cribl.io>.

## License
---
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).
