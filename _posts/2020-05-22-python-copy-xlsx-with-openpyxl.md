---
title: Copy Excel xlsx with openpyxl and print it in Python
categories: [tips, operations, automation]
tags: [operations, automation, tips]
description: Use openpyxl to copy a xlsx and print to printer
---

## What is my requirement?

Due to the COVID19 pandemic my son has to stay at home. So I give him some math quiz compiled by myself for practice. Like addition, substraction, etc. I used MS Excel for the layout and print and I used Python to generate some quiz as well. Each time I generate the quiz I had to manually import in Excel to keep the format and layout in order to fill in one print page. But I was tired with that so I searched online to find the new way.

Unfortunately there is no one module can manipulate the xlsx directly. I got some ideas and decided to copy from template and update the value in new file and try to print it automatically.

## Copy format from template and generate the new file

{% highlight python %}
import openpyxl
from openpyxl import load_workbook, Workbook

infile = load_workbook('../tmp/tmpl.xlsx')
outfile = Workbook()
new_sheet = outfile.active
default_sheet = infile.active

from copy import copy

for row in default_sheet.rows:
    for cell in row:
        #new_cell = new_sheet.cell(row=cell.row, column=cell.col_idx,
        new_cell = new_sheet.cell(row=cell.row, column=cell.column,
                value= cell.value)
        if cell.has_style:
            new_cell.font = copy(cell.font)
            new_cell.border = copy(cell.border)
            new_cell.fill = copy(cell.fill)
            new_cell.number_format = copy(cell.number_format)
            new_cell.protection = copy(cell.protection)
            new_cell.alignment = copy(cell.alignment)
# copy the height and width
for rowno in range(1, 13):
    new_sheet.row_dimensions[rowno].height = default_sheet.row_dimensions[rowno].height
for col in 'ABCDE':
    new_sheet.column_dimensions[col].width = default_sheet.column_dimensions[col].width
# the merged file was not copied
new_sheet.merge_cells('A1:E1')
# save to new file
outfile.save('../tmp/out.xlsx')
{% endhighlight %}

The code is a simple update from [Stackoverflow](https://stackoverflow.com/questions/23332259/copy-cell-style-openpyxl), based on Charlie's answer

## Print the file

I'm trying to print on **Windows 10** so to code is:

{% highlight python %}

import win32api
import win32print

win32api.ShellExecute (
  0,
  "print",
  r"C:\fc\local\tmp\out.xlsx",
  '/d:"%s"' % win32print.GetDefaultPrinter (),
  ".",
  0
)
{% endhighlight %}

This is from [this link](http://timgolden.me.uk/python/win32_how_do_i/print.html)

For **Linux** platform you can find at [this stackoverflow page](https://stackoverflow.com/questions/12723818/print-to-standard-printer-from-python), at the answer from Anuj