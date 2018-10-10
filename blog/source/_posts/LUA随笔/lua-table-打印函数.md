---
title: lua table 打印函数
date: 2018-06-20 10:04:45
tags:
categories: LUA随笔
---
## 函数来源
函数来自 [print_lua_table](https://gist.github.com/rangercyh/5814003)
以后会根据自己对LUA的理解，写出更完美的打印函数
## 函数内容，已根据理解优化部分
```lua
function luaPrint (value, dislefttab)
	local key,v
	dislefttab = dislefttab or 0

	for key, v in pairs(value) do
		if type(key) == "string" then
			key = string.format("%q", key)
		end
		local szSuffix = ""
		if type(v) == "table" then
			szSuffix = "{"
		end
		local szPrefix = string.rep("\t", dislefttab)
		formatting = szPrefix.."["..key.."]".." = "..szSuffix
		if type(v) == "table" then
			print(formatting)
			luaPrint(v, dislefttab + 1)
			print(szPrefix.."}")
		else
			local szValue = ""
			if type(v) == "string" then
				szValue = string.format("%q", v)
			else
				szValue = tostring(v)
			end
			print(formatting..szValue)
		end
	end
end
```