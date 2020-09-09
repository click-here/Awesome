AWK isn't meant for parsing CSV files but they are a good file format to show basics of AWK.  In fact, I'm going to reference other tools for doing things that you might want to do while working with csv's and awk.

I downloaded the Single Audit [general file](https://www2.census.gov/pub/outgoing/govs/singleaudit/general.zip) data set from [FAC](https://facweb.census.gov/).  They currently delimit this file with pipes but have done comma's in the past. Adjust accordingly if following along and changes have occured.

 ```bash
 $ csvcut -n general.txt
 ```

Out:
```
  1: AUDITYEAR
  2: DBKEY
  3: TYPEOFENTITY
  4: FYENDDATE
  5: AUDITTYPE
  6: PERIODCOVERED
  7: NUMBERMONTHS
  8: EIN
  9: MULTIPLEEINS
 10: EINSUBCODE
 11: DUNS
 12: MULTIPLEDUNS
 13: AUDITEENAME
 14: STREET1
 15: STREET2
 16: CITY
 17: STATE
 18: ZIPCODE
 etc...
 ```

 So, let's just look at columns 1, 8, and 17 using awk.

  ```bash
  awk -F\| '{print $1, $8, $17}' general.txt | head
  ```
  Notice how we need to escape our pipe delimiter in the `-F` flag.

  Out:
  ```
  AUDITYEAR EIN STATE
1997 730776899 OK
1998 730776899 OK
1999 730776899 OK
2000 730776899 OK
2001 730776899 OK
2002 730776899 OK
2003 730776899 OK
2004 730776899 OK
2005 730776899 OK
```

Let's answer the question, "How many rows per State?" Let's also use the zipped file.

 ```bash
zcat general.zip | awk -F\| '{print $17}' | sort | uniq -c
 ```

Out:
```
   6926 AK
  14844 AL
  13478 AR
    103 AS
  13477 AZ
  79233 CA
  11898 CO
  10936 CT
   9764 DC
   1991 DE
   etc...
  ```

Let's save Audit Years >= 2013 to a csv file.

  ```bash
  zcat general.zip | awk -F\| '{OFS=",";$1=$1} $1>2013 {print $0}' > greater2013.txt
  ```
  Notice, we set the "Output Field Seperator to a comma. Also, take a look at [$1 = $1   # force record to be reconstituted](https://www.gnu.org/software/gawk/manual/html_node/Changing-Fields.html)

  Let's list some files in an s3 bucket and save the result as a csv file. Default seperator is whitespace for `aws s3 ls` so we need to change it with awk.

```bash
aws s3 ls s3://bucket/prefix/ --recursive | awk 'BEGIN {OFS=","} {$1=$1;print $0}' > prefix.csv
  ```

What if you wanted to export a dynamodb table to csv?

```bash
aws dynamodb scan \
    --table-name table_name \
    --output text \
    --query "Items[*].[col1.S,col2.S,col3.S,]" | \
    awk 'BEGIN { OFS=","; print "col1","col2","col3"} {OFS=","; $1=$1; print $0}' \
    > dynamodb_table.csv
```