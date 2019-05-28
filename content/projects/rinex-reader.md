+++ 
draft = true
date = 2014-12-06T13:54:40-04:00
title = "RINEX Reader"
slug = "rinex" 
katex = "true"
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


This was my first major project with python, most of this post will be taken from the assignment [submission](https://github.com/sonalranjit/RINEX/blob/master/Final%20GPS%20Project/ESSE%204610%20Project.pdf). The project was part of my GPS course in Undergrad, the project involved creating a python program to get an epoch by epoch least squares position solution of a GPS receiver using it's observation and navigation RINEX files. This project was done with in part with Patrick Lasagne.


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

#### Data section
The data section contains the epoch by epoch observations by the receiver to the satellite. It contains measurements for several satellites, the measurements included are limited by what is present in the 'OBS' observation list mentioned above. The reader reads the first line, which has the epoch, the total number of satellites and the list of the satellites. The epoch is made into a key in the form 'YY:MM:DD:HH:MM:S.sssssss', it maps to a dictionary that will be used to include all satellite observations. The reader then uses the number of satellites present and the number of observations expected to determine how many lines remain in the observation block. It then creates a list of the satellites andd uses each three-digit PRN value as a key in the dictionary paired with the epoch key mentioned above. The reader reads line by line and creates a dictionary that is matched to the correct PRN key for each satellite PRN. The dictionaries referred to by the PRN keys are then filled with key value pairs that refer to the observation information stored in the observation block. While all the information was stored in the observation dictionary only some of it was used, the key descriptions and the dictionary structure can be seen below.

|**Key**|**Value**|**Description**|
|-------|---------|---------------|
|'YY:MM:DD:HH:MM:S.sssssss|Dictionary with sub keys 'PRN','PRN','PRN', and 'LIST', 'NUMSAT'|The key refers to the observation block epoch. It mapps to a dictionary which contains dictionaries referred to by satellite PRN numbers, each dictionary corresponds to a different satellite in the observation block. 'LIST' maps to a list of all satellite PRN numbers (for iterating) 'NUMSAT' refers to the integer number of satellites.|
|'PRN'|Dictionary with sub keys 'C1', 'P1', 'P2', etc|Contains the observations for the given satellite at the given epoch.|

**Dictionary Structure**
```python
{
    '07:1:1:0:0:0.0000000':{
        'G06':{
            'C1':20857582.237,
            'P1':20857582.016,
            'P2':20857580.516,
            },
        'G07':{
            'C1':20418227.176,
            'P1':20418226.502,
            'P2':20418225.415
            },
        'G10':{
            'C1':23445664.779,
            'P1':23445663.158,
            'P2':23445663.052,
        },
        'LIST':['G06','G10','G07'],
        'NUMSAT':10
    },
    '07:1:1:0:0:30.0000000':{
        'G06':{
            'C1':20868360.224
            ...
        }
    }
}
```

### Navigation File Reader

#### Header Section
The header file contains metadata information about the RINEX file such as the RINEX version. The line identifier in header has reserved space of 20 characters at the end of the line. Each RINEX file has 80 characters per line therefore the location of the header labels are always known. For each header label the location of the variables is defined by the index of the character spacing. For each label a dictionary key is defined as the variable value corresponding to the label is stored. Beisdes the RINEX version the header contains comments about the RINEX file, and 3 optional information such as the emphemeris coefficients of the Ionosphere *alpha* and *beta*. Also optionally included in the header is the clock offset coefficients of UTC from GPS and the number of leap seconds the current GPS time is from UTC. The end of the header is defined by the label 'END OF HEADER' once this is reached then parsing of the navigation data begins.

#### Data Section
The navigation data is sorted in the RINEX file by the GPS PRN number then epoch the ephemeris data is valid for. Again the spacing for each element of the broadcast is defined for every line. The label for the PRN key is defined for the first 2 space of the line after the header. Once the PRN number is parsed then the lines are counted from that point on. Each element of the line has certain index and spacing, then the data is parsed until the 8th line is reached and a new PRN number is parsed. Each broadcast contains 29 variables that the satellite broadcasts. These 29 variables are given in a block of 8 lines and 4 columns, one of the spaces is for the PRN number and the epoch while the other 2 spaces of the 32 spaces are left empty in the RINEX 2.11 version.

The data is collectively stored with a nestedd dictionary, where the top most key is the GPS PRN number, then the data blocks are divided by subkeys of the epoch that the broadcast data is valid for.

|**Key**|**Value**|**Description**|
|-------|---------|---------------|
|'PRN'|Dictionary with sub keys epoch|Contains the 29 navigation parameters and broadcasted navigation information by the satellite.|

**Navigation Dictionary Structure**
```python
{
    'GO1':{'10:1:1:16:0:0.0':{
        'Cic':5.587935447693e-09,
        'Cis':7.636845111847e-08,
        'Crc':183.21875,
        'Crs':-165.46875,
        'Cuc':-8.651986718178e-06,
        'Cus':1.038610935211e-05,
        'Deln':5.447369761959e-09,
        'Ecc':0.003840734250844,
        'Fit_int':4.0,
        'GPS_W':1564.0,
        'IDOT':-7.000291590243e-11,
        'IODC':51.0,
        'IODE':51.0,
        'Io':0.9635689853388,
        'L2_CC':1.0,
        'L2_P':0.0,
        'Mo':0.0841761497488,
        'OMEGA':-0.1005044703918,
        'OMEGA_DOT':-7.951402636407e-09,
        'Omega':0.9428572252394,
        'SV_Acc':2.0,
        'SV_CLB':-7.538078352809e-05,
        'SV_CLD':-3.069544618484e-12,
        'SV_CLR':0.0,
        'SV_Health':63.0,
        'SqrtA':5153.668302536,
        'TGD':-1.909211277962e-08,
        'TOE':489600.0,
        'Trans_time':487788.0
    }}
}
```

#### Satellite Position
To compute the receiver coordinates, first the coordinates of the satellite must be determined. It is a fairly straightforward process of determining the satellite coordinates, the Keplerian elements of the satellite is broadcasted in the Navigation file. Each satellite in the GPS constellation does this broadcast of all the GPS satellites in the constellation and broadcasts its position prediction in orbit for each two hours. Once all the 29 parameters in the RINEX navigation file are read the process of determining the satellite position occurs in the module [`satpos.py`](https://github.com/sonalranjit/RINEX/blob/master/Final%20GPS%20Project/SatPos.py)

The process of determining the satellite positions are outlined in detail in ICD [document](https://www.gps.gov/technical/icwg/IS-GPS-200J.pdf). The algorithm is defined in section 20.3.3.4.3 **User Algorithm for Emphemeris Determination** Pages 103-105. The equations are defined in Table 20-IV of the ICD document.

The `SatPos` function strictly follows the defined guidelines, except a few tweaks for the time variable used in the process of determining the satellite position is in GPS seconds of the week, which is the number of seconds that have passed since the start of a GPS week which is on Saturday midnight. The eccentric anomaly defined in the algorithm is solved using an iterative process, the Eccentric anomaly is calculated until the difference of the current value and the previous value are smaller than a defined threshold which in the software was defined to be 0.000000000001. Once this condition is satisfied the current Eccentric anomaly is saved and used for further calculations.
Once all of the Orbital parameters are determined and the position of the satellite in the orbit determined the final output of the function is the ECEF coordinates of the satellite in interest.

### Corrections

#### Ionospheric Correction
The effect due to the ionosphere can be mitigated in two ways. If using a single frequency receiver, the delay caused by the ionosphere can be modelled with the Klobuchar model. Since the software does not attempt to adjust a single frequency receiver observations, the Klobuchar model will not be explained here. When using a ddual frequency receiver, the delay due to the ionosphere can be nearly mitigated with a linear combination of the two equations, one on each frequency. The equation for the Ionofree linear combination that removes most of the ionospheric effect is:

Inline math looks like this  

```tex
This is text with inline math $\sum_{n=1}^{\infty} 2^{-n} = 1$
```