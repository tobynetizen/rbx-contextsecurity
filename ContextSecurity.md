## > ContextSecurity.md
### Introduction
While I was developing my FPS framework for Roblox, I was trying to think of ways to communicate between client and server scripts without using RemoteEvents/RemoteFunctions. I thought of this conceptual transmission protocol using module scripts which is not an applicable solution by any means; However, I thought I would archive it here because it is still useful information to maybe implement in another project.

### Protocol
The system consists of one module script, one local script, and one server script. The location of each does not matter except the module script which must be accessible to both the server and the client. I will first share the module script and them explain how it all works.

```lua
local GlobalContext = setmetatable({}, {})
local ServerContext = setmetatable({}, {})
local LocalContext = setmetatable({}, {})

local ContextType = {
    ["Global"] = "ModuleScript",
    ["Server"] = "Script",
    ["Local"] = "LocalScript"
}

getmetatable(LocalContext).__index = function(tb1, idx)
    return rawget(GlobalContext, idx)
end

getmetatable(LocalContext).__newindex = function(tb1, idx, val)
    rawset(GlobalContext, idx, val)
end

getmetatable(ServerContext).__index = function(tb1, idx)
    return rawget(GlobalContext, idx)
end

getmetatable(ServerContext).__newindex = function(tb1, idx, val)
    rawset(GlobalContext, idx, val)
end

function GlobalContext:GetLocalContext()
    if getfenv(2).script.ClassName == ContextType.Local then
        return LocalContext
    end
end

function GlobalContext:GetServerContext()
    if getfenv(2).script.ClassName == ContextType.Server then
        return ServerContext
    end
end

return GlobalContext
```

Okay, so let's analyze the first part...
```lua
local GlobalContext = setmetatable({}, {})
local ServerContext = setmetatable({}, {})
local LocalContext = setmetatable({}, {})
```
The module contains three tables which are containers for data transmitted. The GlobalContext acts as a gateway for the client and the server to fetch each individual container *(referred to as context)* and is dynamically accessible by requiring it into their script environments.
```lua
local ContextType = {
    ["Global"] = "ModuleScript",
    ["Server"] = "Script",
    ["Local"] = "LocalScript"
}
```
Each script type is mapped to each context which disallows the client accessing the server context *(and vice versa)* as seen below.
```lua
function GlobalContext:GetLocalContext()
    if getfenv(2).script.ClassName == ContextType.Local then
        return LocalContext
    end
end

function GlobalContext:GetServerContext()
    if getfenv(2).script.ClassName == ContextType.Server then
        return ServerContext
    end
end
```
When the GlobalContext is invoked to be localized in either the server or client script environments, it is compared against the class name of the calling environment.
```lua
getmetatable(LocalContext).__index = function(tb1, idx)
    return rawget(GlobalContext, idx)
end

getmetatable(LocalContext).__newindex = function(tb1, idx, val)
    rawset(GlobalContext, idx, val)
end

getmetatable(ServerContext).__index = function(tb1, idx)
    return rawget(GlobalContext, idx)
end

getmetatable(ServerContext).__newindex = function(tb1, idx, val)
    rawset(GlobalContext, idx, val)
end
```
And finally, when the localized GlobalContext gets invoked to add or query an element in it's table, it will act as a proxy and invoke the metamethod of the GlobalContext table in the module script.</br>

### Conclusion
Again — this is not functional as is, but it can still be implemented in other ways.</br>
This is just the basis of understanding for the system wholefully.
