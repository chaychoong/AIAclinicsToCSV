# AIA Clinics PDF to CSV

## Introduction

AIA releases a list of panel clinics once a month in the form of a PDF document. It is hard to find the nearest clinics by looking at tables and thankfully Google's MyMaps allows us to build a custom map to visualise the table. But first, we need to convert this list into a CSV or XLSX format.

## Prerequisites

Anaconda/Miniconda with Jupyter Notebook (with openpyxl installed)   
Bluebeam Revu or any other PDF viewers that can export to XLSX

## Convert from PDF to XLSX

Use Bluebeam/your favourite viewer to convert the PDF file to XLSX format.

Save it as `'clinics.xlsx'`

## Preparing the code

Import the `openpyxl` package

``` python
import openpyxl
```

Load workbook and worksheet.

``` python
# load workbook
wb = openpyxl.load_workbook('clinics.xlsx')

# load active worksheet
ws = wb.active
```

## Extracting the list of zones

We want to extract only the relevant rows. Note that the first cell of the row starts with a zone. Let's start by getting a list of zones.

We can build a counter for items in the first column using a dictionary.

We start by creating an empty dictionary. Iterating through the list using `ws.iter_rows()`, we check if the key is inside the dictionary - `dic.get(key)` will return `None` if the key does not exist - if it is, increment the counter, if it isn't, start the counter.

``` python
zonesDic = {}

for row in ws.iter_rows():
    if zonesDic.get(row[0].value) == None:
        zonesDic[row[0].value] = 1
    else:
        zonesDic[row[0].value] = zonesDic[row[0].value] + 1
```

We can then create a list by sorting the dictionary based on the counter. Notice that zone names are all in uppercase letters. We can remove the rest.

``` python
sortedList = sorted(zonesDic, key=zonesDic.get, reverse=True)

for item in sortedList[:]: # don't remove as iterating. Create a copy, iterate through that copy
    if item == None or not item.isupper():
        sortedList.remove(item)
```

By observing the output, we then manually clean up the zone list.

``` python
zones = ['EAST', 'CENTRAL', 'NORTH-EAST', 'NORTH', 'WEST', 'NORTH-WEST', 'CENTRAL-NORTH', 'CENTRAL-EAST', 'CITY', 'CENTRAL-SOUTH', 'SOUTH-WEST', 'CENTRAL-WEST', 'SOUTH']
```

## Output

Note that `csv.writer` works nice with lists. Therefore, we want the relevant rows to be presented as a list of lists (rows of column items).

Iterate through the all the rows of the spreadsheet once again. We check if the first item is in `zones`. If it is, append it into an output list.

``` python
outList = []

for row in ws.iter_rows():
    if row[0].value in zones:
        tmpline = []
        for cell in row:
            tmpline.append(cell.value)
        outList.append(tmpline)
```

## Write the result to a CSV file

Import the CSV package.

```python
import csv
```

Create a new csv file and use the `writerows` function to create the csv.

``` python
myFile = open('clinics.csv', 'w')
with myFile:
    writer = csv.writer(myFile)
    writer.writerows(outList)
```

We are almost done!

## Further modifications

### CSV newlines

Notice that `writerows` creates an empty row between rows. We fix that by specifying the newline parameter.

``` python
myFile = open('clinics.csv', 'w', newline='')
```

### Postal code

We want to add `"Singapore "` to the front of the postal code and ensure that it is 6 digits. To do that, we change the way we iterate through the rows and use the `zfill` function for strings to rightpad the string with 0s.

``` python
for row in ws.iter_rows():
    if row[0].value in zones:
        tmpline = []
        #for cell in row:
        for i in range(len(row)): # iterate by index
            cell = row[i].value
            if i == 5:
                cell = "Singapore " + str(cell).zfill(6) # Amended here
            tmpline.append(cell)
        outList.append(tmpline)
```

### Get the headers

We need the first row to be the header row for it to work nice with MyMaps.

Note that we have a header row on the 11th row (index 10). We extract that as a list (a row of column items).

``` python
headrow = list(ws.rows)[10]
head = []

for cell in headrow:
    head.append(cell.value)
```

We then write in this row before we `writerows` the rest.

``` python
with myFile:
    writer = csv.writer(myFile)
    writer.writerow(head)
    writer.writerows(outList)
```

## Complete code

Refer to the jupyter notebook for the complete code.
