---
title:  "How to parse CSV data in Bash"
author: "Jawed Salim"
date: 2022-10-02 21:04:58 +0600
description : "Setup chatwoot on kubernetes cluster using helm chart"
tags: [Linux, Bash, CSV]
---

A comma-separated values file is a delimited text file that uses a comma to separate values. Each line of the file is a data record. Each record consists of one or more fields, separated by commas.

## Some Sample Data

You can easily generate some sample CSV data, using sites like [Online Data Generator](https://www.onlinedatagenerator.com/). You can define the fields you want and choose how many rows of data you want. Your data is generated using realistic dummy values and downloaded to your computer.

We created a file containing 100 rows of dummy employee information:

- **id**: A simple unique integer value.
- **firstname**: The first name of the person.
- **lastname**: The last name of the person.
- **job-title**: The person’s job title.
- **email-address**: The person’s email address.
- **branch**: The company branch they work in.
- **state**: The state the branch is located in.

Some CSV files have a header line that lists the field names. Our sample file has one. Here’s the top of our file:

```yaml
id,firstname,lastname,job-title,email-address,branch,state
1,Georgia,Welsch,Doctor,Georgia_Welsch6001@nickia.com,Columbus,West Virginia
2,Molly,Clifford,Lecturer,Molly_Clifford2666@joiniaa.com,Moreno Valley,Florida
3,Eduardo,Barrett,Accountant,Eduardo_Barrett934@typill.biz,Detroit,Alabama
4,Tom,Ellison,Biologist,Tom_Ellison2515@joiniaa.com,Tokyo,North Dakota
5,Harvey,Giles,Inspector,Harvey_Giles9110@typill.biz,Moreno Valley,Maine
6,Rick,Butler,Baker,Rick_Butler8563@zorer.org,Saint Paul,Vermont
7,Sienna,Neal,Healthcare Specialist,Sienna_Neal1487@typill.biz,New Orleans,Texas
8,Lily,Armstrong,Software Engineer,Lily_Armstrong2045@infotech44.tech,Zurich,Hawaii
9,Kendra,Addis,IT Support Staff,Kendra_Addis9070@naiker.biz,San Diego,Texas
```

## Parsing Data Form the CSV file

Let’s write a script that will read the CSV file and extract the fields from each record. Copy this script into an editor, and save it to a file called `"field.sh."`

```sh
#! /bin/bash

while IFS="," read -r id firstname lastname jobtitle email branch state
do
  echo "Record ID: $id"
  echo "Firstname: $firstname"
  echo " Lastname: $lastname"
  echo "Job Title: $jobtitle"
  echo "Email add: $email"
  echo " Branch: $branch"
  echo " State: $state"
  echo ""
done < <(tail -n +2 sample.csv)
```

There's quite a bit packed into our little script. Let's break it down.

We're using a `while` loop. As long as the `while` loop condition resolves to true, the body of the `while` loop will be executed. The body of the loop is quite simple. A collection of `echo` statements are used to print the values of some variables to the terminal window.

The `while` loop condition is more interesting than the body of the loop. We specify that a comma should be used as the internal field separator, with the `IFS=","` statement. The `IFS` is an environment variable. The read command refers to its value when parsing sequences of text.

We're using the read command's `-r` (retain backslashes) option to ignore any backslashes that may be in the data. They'll be treated as regular characters.

The text that the read command parses is stored in a set of variables named after the CSV fields. They could just as easily have been named `field1, field2, ... field7` , but meaningful names make life easier.

The data is obtained as the output from the `tail` command. We're using `tail` because it gives us a simple way to skip over the header line of the CSV file. The `-n +2` (line number) option tells `tail` to start reading at line number two.

The `<(...)` construct is called process substitution. It causes Bash to accept the output of a process as though it were coming from a file descriptor. This is then redirected into the `while` loop, providing the text that the read command will parse.

Make the script executable using the `chmod` command. You’ll need to do this each time you copy a script from this article. Substitute the name of the appropriate script in each case.

```sh
chmod +x field.sh
```

When we run the script, the records are correctly split into their constituent fields, with each field stored in a different variable.

```sh
./field.sh
```

```yaml
Record ID: 1
Firstname: Georgia
Lastname: Welsch
Job Title: Doctor
Email add: Georgia_Welsch6001@nickia.com
Branch: Columbus
State: West Virginia

Record ID: 2
Firstname: Molly
Lastname: Clifford
Job Title: Lecturer
Email add: Molly_Clifford2666@joiniaa.com
Branch: Moreno Valley
State: Florida

...
...
```

> Each record is printed as a set of fields.

## Selecting Fields

Perhaps we don't want or need to retrieve every field. We can obtain a selection of fields by incorporating the cut command.

This script is called `"select.sh."`

```sh
#!/bin/bash

while IFS="," read -r id jobtitle branch state
do
  echo "Record ID: $id"
  echo "Job Title: $jobtitle"
  echo " Branch: $branch"
  echo " State: $state"
  echo ""
done < <(cut -d "," -f1,4,6,7 sample.csv | tail -n +2)
```

We've added the `cut` command into the process substitution clause. We’re using the `-d` (delimiter) option to tell `cut` to use commas `","` as the delimiter. The `-f` (field) option tells `cut` we want fields `one, four, six, and seven`. Those four fields are read into four variables, which get printed in the body of the while loop.

This is what we get when we run the script.

```s
./select.sh
```

```yaml
Record ID: 1
Lastname: Welsch
Branch: Columbus
State: West Virginia

Record ID: 2
Lastname: Clifford
Branch: Moreno Valley
State: Florida

...
...
```

> By adding the cut command, we're able to select the fields we want and ignore the ones we don't.

## The csvkit Toolkit

The CSV toolkit `csvkit` is a collection of utilities expressly created to help work with CSV files. You’ll need to install it on your computer.

To install it on Ubuntu, use this command:

```s
sudo apt install csvkit
```

If we pass the name of a CSV file to it, the csvlook utility displays a table showing the contents of each field. The field content is displayed to show what the field contents represent, not as they’re stored in the CSV file.

Let's try csvlook with our problematic "sample2.csv" file.

```s
csvlook sample.csv
```

```s
|  id | firstname  | lastname   | job-title                   | email-address                      | branch          | state          |
| --- | ---------- | ---------- | --------------------------- | ---------------------------------- | --------------- | -------------- |
|   1 | Georgia    | Welsch     | Doctor                      | Georgia_Welsch6001@nickia.com      | Columbus        | West Virginia  |
|   2 | Molly      | Clifford   | Lecturer                    | Molly_Clifford2666@joiniaa.com     | Moreno Valley   | Florida        |
|   3 | Eduardo    | Barrett    | Accountant                  | Eduardo_Barrett934@typill.biz      | Detroit         | Alabama        |
```

All of the fields are correctly displayed. This proves the problem isn’t the CSV. The problem is our scripts are too simplistic to interpret the CSV correctly.

To select specific columns, use the csvcut command. The `-c` (column) option can be used with field names or column numbers, or a mix of both.

Suppose we need to extract the `first and last names, job titles, and email addresses` from each record, but we want to have the name order as `"last name, first name."` All we need to do is put the field names or numbers in the order we want them.

These three commands are all equivalent.

```s
csvcut -c lastname,firstname,job-title,email-address sample.csv
```

```s
csvcut -c lastname,firstname,4,5 sample.csv
```

```s
csvcut -c 3,2,4,5 sample.csv
```

We can add the csvsort command to sort the output by a field. We’re using the `-c` (column) option to specify the column to sort by, and the `-r` (reverse) option to sort in descending order.

```s
csvcut -c 3,2,4,5 sample2.csv | csvsort -c 1 -r
```

To make the output prettier we can feed it through csvlook.

```s
csvcut -c 3,2,4,5 sample2.csv | csvsort -c 1 -r | csvlook
```

A neat touch is that, even though the records are sorted, the header line with the field names is kept as the first line. Once we’re happy we have the data the way we want it we can remove the csvlook from the command chain, and create a new CSV file by redirecting the output into a file.

