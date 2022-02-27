---
title: Provision Route53 DNS Records with the AWS CLI
tags:
  - aws
  - dns
  - route 53
  - how-to
  - aws cli
---

Update, i'm still dedicated to doing as much as I can through the AWS CLI. I've learned a ton this way and it's given me better insight into what's possible with different aws services. Learned a ton about using `sed` as well. Even though the find/replace in vim is nearly identical to sed it took me a few hours of reading the docs and stackoverflow posts about the syntax for regular expressions to really get the hang of it. While sed's an awesome tool to know, next time I think I want to try a [command line tool](https://stedolan.github.io/jq/) that can interact with serialized data more easily. Like everything my writing style is still a work in progress so I'll probably go back and change things around when I've found something I vibe with.

**Note:** This process follows the creation of an A record and assumes the public IP to be associated is already known

## Get Hosted Zone ID
This command searches for hosted zones based on name. e.g. bigkea.com, Output is the hosted zone id.
```bash
dname="<zone-name>" \
aws route53 list-hosted-zones-by-name --output=text | grep $dname | sed -n "s/^.*zone\/\(\S*\).*$/\1/p"
```

## Get the CLI Skeleton
Write some yaml for the request.
```bash
aws route53 change-resource-record-sets --generate-cli-skeleton yaml-input > change-record.yaml
```
That gives us the skeleton. I changed the `Action:` to `UPSERT` (updates record if record exists/inserts new record if record doesn't exist), `Name:` to `jenkins.bigkea.com`, `Type:` to `A`, and `Value:` to my public IP. I deleted unused lines so if you were making a `cname` record, you would change value to `CNAME` and fill out the cname specific lines using an original skeleton.
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
You can confirm the record set was created with:
```bash
aws route53 list-resource-record-sets --hosted-zone-id <zone-id>
```

