---
title: Provision Route53 DNS Records with the AWS CLI
---

I'm still dedicated to doing as much as I can through the AWS CLI while I'm working on these learning projects. I've learned a ton this way and it's giving me better insight to how the services work. I'm also enjoying the process of developing a new writing style. Like everything it's still a work in progress so I'll probably go back and change things around when I've found a style and process I'm happy with.
**Note:** This process follows the creation of an A record and assumes the public IP to be associated is already known

## Get Hosted Zone ID
This command searches for hosted zones based on name. e.g. bigkea.com, Output is the hosted zone id.
```bash
dname="<zone-name>" \
aws route53 list-hosted-zones-by-name --output=text | grep $dname | sed -n "s/^.*zone\/\(\S*\).*$/\1/p"
```

## Get the CLI Skeleton
I may try out a [command line tool](https://stedolan.github.io/jq/) to interact with serialized data soon but until then let's just make some yaml for the request.
```bash
aws route53 change-resource-record-sets --generate-cli-skeleton yaml-input > change-record.yaml
```
That gives us the skeleton. I changed the action to upsert, name to jenkins.bigkea.com, type to A, and public IP to value.
```yaml
HostedZoneId: '<zone-id>'  # [REQUIRED] The ID of the hosted zone that contains the resource record sets that you want to change.
ChangeBatch: # [REQUIRED] A complex type that contains an optional comment and the Changes element.
  Comment: ''  #  Optional.
  Changes: # [REQUIRED] Information about the changes to make to the record sets.
  - Action: UPSERT # [REQUIRED] The action to perform. Valid values are: CREATE, DELETE, UPSERT.
    ResourceRecordSet: # [REQUIRED] Information about the resource record set to create, delete, or update.
      Name: 'jenkins.bigkea.com'  # [REQUIRED] For ChangeResourceRecordSets requests, the name of the record that you want to create, update, or delete.
      Type: A # [REQUIRED] The DNS record type. Valid values are: SOA, A, TXT, NS, CNAME, MX, NAPTR, PTR, SRV, SPF, AAAA, CAA, DS.
      TTL: 0 # The resource record cache time to live (TTL), in seconds.
      ResourceRecords: # Information about the resource records to act upon.
      - Value: '<public-ip>'  # [REQUIRED] The current or new DNS record value, not to exceed 4,000 characters.
```
## Issue the Command
Now finally make the request.
```bash
aws route53 change-resource-record-sets --cli-input-yaml file://record-set.yaml
```
