+++ 
draft = true
date = 2014-12-06T13:54:40-04:00
title = "RINEX Reader"
slug = "rinex" 
tags = [
    "python",
    "GPS",
    "development",
]
categories = [
    "Development",
    "python",
]
+++

[Github Repo](https://github.com/sonalranjit/RINEX)


This was my first major project with python, most of this post will be taken from the assignment [submission](https://github.com/sonalranjit/RINEX/blob/master/Final%20GPS%20Project/ESSE%204610%20Project.pdf). The project was part of my GPS course in Undergrad, the project involved creating a python program to get an epoch by epoch least squares position solution of a GPS receiver using it's observation and navigation RINEX files. 


## Objectives
---
* Develop python functions to process RINEX observations and navigation files and store the data in data structures.
* Develop python function to determine satellite position using the navigation files.
* Develop python functitons to compute and apply ionospheric and tropospheric corrections, as well as satellite offsets, and apply them to observation pseudoranges where applicable.
* Run an epoch by epoch least squares adjustmentt to determine the location and clock offset of the receiver.
* Compute various DOP values using the covariance matrix from the least squares adjustment.
* Determine the accuracy and precision of the solution.
* Explain why the values were acquired.

## Software Structure
---
RINEX stands for Receiver Independent Exchange Format, it is stored in an ascii format and contains GNSS Observations. The two RINEX files that the software reads are the observation and navigation files.

It was decided that the information stored in the observation and navigation files would be stored in a python dictionary data structure. A dictionary holds a set of unique "key-value" pairs that allow information to be accessed rapidly, without having to deal with messy array indices. The implementation is not important, all that must be kept in mind is that dictionaries allow the information present in the two RINEX files to be stored and later rapidly accessedd with meaningdful keywords at a later date. The only downside of dictionaries is that there is no reliable way to iterate through a dictionary in a certain order, to counteract this, each dictionary that may need to be iterated over also includes a list of keys corresponding to the key 'LIST' in the order they were placed into the dictionary.

The parsing of the RINEX navigation file is done line by line and character by character. Each element in the RINEX file is given a certain amount of spacing to fill up. This information on the spacing is found in the GAGE GROUP's gLab RINEX file format [description](https://gage.upc.es/sites/default/files/gLAB/HTML/GPS_Navigation_Rinex_v4_CNAV.html)

### Observation File Reader

The observation RINEX file contains all of the GPS receiver's observations that it has collected over it's setup time. The file is split into two components, the header, which contains important information about the receiver itself, and the data section, which contains phase and pseudorange observations to various satellites at various epochs.

#### Header Section
The header section contains information relating to the receiver, such as it's position in a given coordinate system, it's antenna height, or it's make and model. It also contains clerical information pertaining to the observation set, and may include extra information present in comments. Most importantly, the header details the type of observations present in the RINEX file.

The header contains information for the first 60 characters (there are 80 on a line), the remaining 20 consitute a label which explains what is detailed in the previous 60 characters, each line is then terminated witth a new line break (\n). To fully parse the header, the RINEX file is read in line by line. At each line, the label is checked to see what kind of header information is present on that line, this continues until END OF HEADER was reached, at that point, the software would switch from trying to read the header, to trying to read the observations. While the software attempts to read the header, it consults the last word of the label, and attempts to match it with a predefined list present in the software (with HTYPER method). In the event that two or more label types have the same last word, then the second last word is used instead. Once the label is identified, the software parses the line according to it (with ASSIGNDIC). The values present are extracted from the line with direct indexing (ex, character 0-16), are assigned meaningful "keys", and are placed in the dictionary. In th event that a key already exists (or a multi-line label is reached), the value at the key is changed depending on label type. For example, if a COMMENT label is reached, the current string value stored at the key 'COMMENT' has the new value appended to it's end. In the event # / TYPES OF OBSERV occurs on more than two lines (with more than nine observation types), the list that holds the observation types has the extra values appended to it. While all information in the header section was stored in the obsHead dictionary, only a few important keys present in the header are used in the software, they are:

|*Key*     |*Value*                                       |*Description*                                              |
|----------|----------------------------------------------|-----------------------------------------------------------|
|'POS'     |Dictionary with sub keys 'X', 'Y', and 'Z'    |Contains the X, Y, and Z position of the receiver.         |
|'OBSTYP'  |Dictionary with sub keys 'NUM' and 'OBS'      |'NUM' contains the integer number of satellites 'OBS' contains a list of the two alphanumeric GPS observation types (ie 'C1', 'P1')|