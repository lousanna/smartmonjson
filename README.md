## Smartmonjson  

This is a clone of smartmontools with the added option of supplying a --json argument to print disk info and attribute info in json.  

## How to Use

`cd smartmontools`  
`./autogen.sh`  
`./configure`  
`make`  
`sudo ./smartctl --json /dev/sda`  

## Relevant files changed  

1. smartctl.cpp 
2. ataprint.cpp  
3. ataprint.h  
