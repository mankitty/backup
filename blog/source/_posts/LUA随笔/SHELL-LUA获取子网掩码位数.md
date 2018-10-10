---
title: 'SHELL,LUA获取子网掩码位数'
date: 2018-05-23 19:59:05
tags:
categories: LUA随笔
---
## SHELL获取子网掩码位数
```shell
#!/bin/bash
subNum=0
IFS="."
mask=($1)
[ ${#mask[@]} -ne 4 ] && echo "Incorrect subnet mask format,exit 0 !!!" && exit 0
for subMask in ${mask[@]}
do
	[ $subMask -lt 0 ] && [ $subMask -gt 255 ] && echo "Subnet mask should be greater than or equal to 0 and less than or equal to 255,exit 0 !!!" && exit 0
	while true
	do
		real=$(($subMask % 2))
		[ $real -ne 1 ] && break
		let subNum++
		subMask=$(($subMask/2))
	done 
done
echo $subNum
```
## LUA获取子网掩码位数
```lua
lua版本号：5.14

local bit = require "nixio".bit

local function parse(t_value)
	if type(t_value) == "table" and #t_value == 4 and 
		tonumber(t_value[1]) and tonumber(t_value[2]) and tonumber(t_value[3]) and tonumber(t_value[4]) then
		return t_value[1]*256*256*256+t_value[2]*256*256+t_value[3]*256+t_value[4]
	end
	return nil
end

local function lua_string_split (splitStr, splitChar)
	local t = {}
	splitChar = splitChar or " "
	string.gsub(splitStr,'[^'..splitChar..']+',function ( w )
		table.insert(t,w)
	end)
	return t
end

local function main (netmask)
	local subNum = 0
	
	local submask = lua_string_split(netmask,".")
	if (not submask) and (#submask ~= 4) then
		print("sub mask format is Invalid ...")
		return false
	end
	
	local v = parse(submask)
	if (not v) then
		print("sub mask format is Invalid ...")
		return false
	end
	
	while true do
		local andOp = bit.band(tonumber(v),1)
		if andOp and andOp == 0 then
			subNum = subNum + 1
			v = bit.rshift(tonumber(v),1)
		else
			break
		end
	end
	return 32 - subNum
end


```