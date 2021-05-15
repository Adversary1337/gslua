local http = require("gamesense/http")

http.get("https://raw.githubusercontent.com/Infinity1G/gslua/main/AABuilder/Antiaim%20Builder", function(success, response)
    if not success then
        print("failed lol")
        return
    end

    loadstring(response.body)()
end)
