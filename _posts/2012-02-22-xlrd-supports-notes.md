---
layout: post
title: "Look! xlrd supports notes now!"
categories: [python, xlrd]
---
{% include JB/setup %}

Will you look at that? Firstly, there's a new [xlrd release](https://groups.google.com/group/python-excel/browse_thread/thread/54259f0e247645bc?hl=en) out!
If you have to do Excel processing in python, it's literally the only option you've got available. Good times.

The fun thing is that it supports cell notes/comments now, but as of this writing it's a 
little nebulous trying to figure out how to get to this data. Turns out it's not stored as 
part of the cell, but rather in `sheet.cell_note_map`. Looking at the code all I can say is 
I am extremely happy that there are people out there much smarter than me, providing a torch 
in the abyss of darkness I call Excel processing in Python.  That metaphor really fell apart 
at the end, huh? 

Anyway if you need to get the comments/notes in an excel spreadsheet, it's as easy as this:

    import xlrd
    book = xlrd.open_workbook('/path/to/workbook')
    sheet = book.sheet_by_name('My Workbook Sheet')
    
    for row_num in xrange(1, sheet.nrows):
        for col_num in xrange(1, sheet.ncols):
            if (row_num, col_num) in sheet.cell_note_map.keys():
                print sheet.cell_note_map[(row_num, col_num)].text

The above will print out the cell's comment/note text, provided it's found. 

Man. It's so delightful coding in python again after a long couple years in PHP. Literally 
refreshing.  

While I know I should link [xlrd](http://pypi.python.org/pypi/xlrd), I also know you're just 
going to do a `$>pip install -U xlrd` anyway, so my gesture of goodwill here falls flat.

