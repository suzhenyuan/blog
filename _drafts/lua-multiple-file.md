Spreading a Lua Lapis application across multiple files

* app.lua
local app = lapis.Application()
package.loaded.app = app

require "app_1"
require "app_2"

return app

* app_1.lua
local app = require "app"

app:get("/hello", function(self) end)
app:post("/world", function(self) end)

* app_2.lua
local app = require "app"

app:get("/user/login", function(self) end)
app:post("/user/logout", function(self) end)