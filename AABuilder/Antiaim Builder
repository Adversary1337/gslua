-- #region required liraries
local clipboard = require("gamesense/clipboard") or error("Clipboard API is required")
-- #endregion

-- #region api/references/data
local client_delay_call, client_key_state, client_screen_size, client_set_event_callback, client_unset_event_callback, client_userid_to_entindex, database_read, database_write, entity_get_local_player, entity_get_prop, entity_hitbox_position, entity_is_dormant, entity_is_enemy, globals_curtime, globals_realtime, math_abs, math_floor, math_sqrt, renderer_gradient, renderer_indicator, renderer_measure_text, renderer_text, table_remove, ui_get, ui_is_menu_open, ui_new_button, ui_new_checkbox, ui_new_combobox, ui_new_colorpicker, ui_new_hotkey, ui_new_listbox, ui_new_multiselect, ui_new_slider, ui_new_textbox, ui_reference, ui_set, ui_set_callback, ui_set_visible, ui_update = client.delay_call, client.key_state, client.screen_size, client.set_event_callback, client.unset_event_callback, client.userid_to_entindex, database.read, database.write, entity.get_local_player, entity.get_prop, entity.hitbox_position, entity.is_dormant, entity.is_enemy, globals.curtime, globals.realtime, math.abs, math.floor, math.sqrt, renderer.gradient, renderer.indicator, renderer.measure_text, renderer.text, table.remove, ui.get, ui.is_menu_open, ui.new_button, ui.new_checkbox, ui.new_combobox, ui.new_color_picker, ui.new_hotkey, ui.new_listbox, ui.new_multiselect, ui.new_slider, ui.new_textbox, ui.reference, ui.set, ui.set_callback, ui.set_visible, ui.update
local clipboard_get, clipboard_set = clipboard.get, clipboard.set

local references = {
	pitch = ui_reference("AA", "Anti-aimbot angles", "Pitch"),
	yawbase = ui_reference("AA", "Anti-aimbot angles", "Yaw base"),
	yaw = {ui_reference("AA", "Anti-aimbot angles", "Yaw")},
	yawjitter = {ui_reference("AA", "Anti-aimbot angles", "Yaw jitter")},
	bodyyaw = {ui_reference("AA", "Anti-aimbot angles", "Body yaw")},
	freestandbodyyaw = ui_reference("AA", "Anti-aimbot angles", "Freestanding body yaw"),
	lbytarget = ui_reference("AA", "Anti-aimbot angles", "Lower body yaw target"),
	fakelimit = ui_reference("AA", "Anti-aimbot angles", "Fake yaw limit"),
	edgeyaw = ui_reference("AA", "Anti-aimbot angles", "Edge yaw"),
	freestanding = {ui_reference("AA", "Anti-aimbot angles", "Freestanding")}
}

local main_table = {"[»] Stage builder", "[»] Configs", "[»] Indicators", "----------------------------------------------"}
local main_data = {}
local init_size = #main_table
for i = 1, init_size do main_data[#main_data+1] = "le nothing" end -- keep main_table and main_data the same length
local configs = database_read("aabuilder_configs") or {}
local should_switch = false
local can_switch = true
local can_switch_menu = true
local last_switch = 0
local current_stage = init_size+1
local brute_last_miss = 0
local brute_timer = 0
local timer = true
local timer_timer = 0
local sw, sh = client_screen_size()
local menu = {"LUA", "A", "B"}

local cache = database_read("aabuilder_cache") or {}
for i = 1, #cache do
	main_table[#main_table+1] = cache[i].tab
	main_data[#main_data+1] = cache[i].data
end
-- #endregion

-- #region ui elements
local main_master_switch = ui_new_checkbox(menu[1], menu[2], "AABuilder master switch")
local main_listbox = ui_new_listbox(menu[1], menu[2], "Main listbox", main_table)

local stage_name = ui_new_textbox(menu[1], menu[2], "Current stage name")
local stage_add = ui_new_button(menu[1], menu[2], "Add stage", function() end)
local stage_duplicate = ui_new_button(menu[1], menu[2], "Duplicate", function() end)
local stage_remove = ui_new_button(menu[1], menu[2], "Remove", function() end)
local stage_moveup = ui_new_button(menu[1], menu[2], "Move up", function() end)
local stage_movedown = ui_new_button(menu[1], menu[2], "Move down", function() end)

local config_listbox = ui_new_listbox(menu[1], menu[3], "Config listbox", configs)
local config_name = ui_new_textbox(menu[1], menu[3], "Config name")
local config_save = ui_new_button(menu[1], menu[3], "Save", function() end)
local config_load = ui_new_button(menu[1], menu[3], "Load", function() end)
local config_remove = ui_new_button(menu[1], menu[3], "Remove", function() end)
local config_import = ui_new_button(menu[1], menu[3], "Import from clipboard", function() end)
local config_export = ui_new_button(menu[1], menu[3], "Export to clipboard", function() end)

local ind_selector = ui_new_combobox(menu[1], menu[2], "Indicate stages", {"Off", "Indicator", "Array"})
local ind_selector_color = ui_new_colorpicker(menu[1], menu[2], "Indicate stages color", 255, 255, 255, 200)

local stage_switch = ui_new_combobox(menu[1], menu[3], "Switch to next stage after", {"Timer", "Bruteforce", "Weapon fire", "Hotkey"})
local stage_timer = ui_new_slider(menu[1], menu[3], "Time to next stage", 1, 2000, 100, true, "ms")
local stage_hotkey = ui_new_hotkey(menu[1], menu[3], "Stage switch hotkey", true)
local stage_pitch = ui_new_combobox(menu[1], menu[3], "Pitch", {"Off", "Default", "Down", "Up", "Minimal", "Random"})
local stage_yawbase = ui_new_combobox(menu[1], menu[3], "Yaw base", {"Local view", "At targets"})
local stage_yaw = ui_new_combobox(menu[1], menu[3], "Yaw", {"Off", "180", "Spin", "Static", "180 Z", "Crosshair"})
local stage_yawval = ui_new_slider(menu[1], menu[3], "\n Yaw slider", -180, 180, 0, true)
local stage_yawjitter = ui_new_combobox(menu[1], menu[3], "Yaw jitter", {"Off", "Offset", "Center", "Random"})
local stage_yawjitterval = ui_new_slider(menu[1], menu[3], "\n Yaw jitter slider", -180, 180, 8, true)
local stage_bodyyaw = ui_new_combobox(menu[1], menu[3], "Body yaw", {"Off", "Static", "Jitter", "Opposite"})
local stage_bodyyawval = ui_new_slider(menu[1], menu[3], "\n Body yaw slider", -180, 180, 60, true)
local stage_freestandingbodyyaw = ui_new_checkbox(menu[1], menu[3], "Freestanding body yaw")
local stage_lbytarget = ui_new_combobox(menu[1], menu[3], "Lower body yaw target", {"Off", "Sway", "Opposite", "Eye yaw"})
local stage_fakelimit = ui_new_slider(menu[1], menu[3], "Fake yaw limit", 0, 60, 60, true)
local stage_edgeyaw = ui_new_checkbox(menu[1], menu[3], "Edge yaw")
local stage_freestanding = ui_new_multiselect(menu[1], menu[3], "Freestanding", {"Default"})
local stage_freestanding_hotkey = ui_new_hotkey(menu[1], menu[3], "Freestanding hotkey", true)
-- #endregion

-- #region helper functions
local function clear_stage_data()
	for i = init_size+1, #main_table do
		table_remove(main_table, init_size+1)
		table_remove(main_data, init_size+1)
	end
end

local function copy_stage_data(index)
	local data = {}

	data = {
		switchtype = 		index.switchtype,
		timer = 			index.timer,
		pitch = 			index.pitch,
		yaw = 				index.yaw,
		yawval = 			index.yawval,
		yawjitter = 		index.yawjitter,
		yawjitterval = 		index.yawjitterval,
		bodyyaw = 			index.bodyyaw,
		bodyyawval = 		index.bodyyawval,
		freestandbodyyaw = 	index.freestandbodyyaw,
		lbytarget = 		index.lbytarget,
		fakelimit = 		index.fakelimit
	}

	return data
end

local function GetClosestPoint(A, B, P) -- this is stolen because math is hard
	local a_to_p = { P[1] - A[1], P[2] - A[2] }
	local a_to_b = { B[1] - A[1], B[2] - A[2] }

	local atb2 = a_to_b[1]^2 + a_to_b[2]^2

	local atp_dot_atb = a_to_p[1]*a_to_b[1] + a_to_p[2]*a_to_b[2]
	local t = atp_dot_atb / atb2

	return { A[1] + a_to_b[1]*t, A[2] + a_to_b[2]*t }
end
-- #endregion

-- #region other functions
local function on_bullet_impact(e) -- stolen from some bruteforce lua, im way too lazy to do this myself
	if main_data[current_stage] == nil then
		return
	end

	if main_data[current_stage].switchtype ~= "Bruteforce" then
		return
	end

	local ent = client_userid_to_entindex(e.userid)
	if not entity_is_dormant(ent) and entity_is_enemy(ent) then
		local ent_shoot = { entity_get_prop(ent, "m_vecOrigin") }
		ent_shoot[3] = ent_shoot[3] + entity_get_prop(ent, "m_vecViewOffset[2]")
		local player_head = { entity_hitbox_position(entity_get_local_player(), 0) }
		local closest = GetClosestPoint(ent_shoot, { e.x, e.y, e.z }, player_head)
		local delta = { player_head[1]-closest[1], player_head[2]-closest[2] }
		local delta_2d = math_sqrt(delta[1]^2+delta[2]^2)

		if math_abs(delta_2d) < 40 and globals_curtime() - brute_last_miss > 0.12 then
			should_switch = true
			brute_timer = globals_curtime() + 5
			brute_last_miss = globals_curtime()
		end
	end
end

local function on_prestart()
	if current_stage > #main_data then
		current_stage = init_size + 1
	end

	local curdata = main_data[current_stage]

	if curdata.switchtype == "Bruteforce" then -- reset bruteforce on round start
		current_stage = init_size + 1
	end
end

local function on_connect_full()
	last_switch = 0
	brute_last_miss = 0
    current_stage = init_size + 1
end

local function on_weapon_fire(e)
	if client_userid_to_entindex(e.userid) ~= entity_get_local_player() then
		return
	end

	if current_stage > #main_data then
		current_stage = init_size + 1
	end

	local curdata = main_data[current_stage]

	if curdata.switchtype ~= "Weapon fire" then
		return
	end

	should_switch = true
end

local function on_shutdown()
	local temp = {}
	for i = init_size+1, #main_table do
		temp[#temp+1] = {
			tab = main_table[i],
			data = main_data[i]
		}
	end

	database_write("aabuilder_cache", temp)
	database_write("aabuilder_configs", configs)
end
-- #endregion

-- #region handle functions 
local function handle_visibility()
	local main = ui_get(main_master_switch)
	local cur = ui_get(main_listbox)+1
	local curdata = main_data[cur]
	local bool_stage = main and cur > init_size and #main_data > init_size

	if bool_stage and curdata ~= nil then
		ui_set(stage_switch, 				curdata.switchtype)
		ui_set(stage_timer, 				curdata.timer)
		ui_set(stage_pitch, 				curdata.pitch)
		ui_set(stage_yaw, 					curdata.yaw)
		ui_set(stage_yawval, 				curdata.yawval)
		ui_set(stage_yawjitter, 			curdata.yawjitter)
		ui_set(stage_yawjitterval, 			curdata.yawjitterval)
		ui_set(stage_bodyyaw, 				curdata.bodyyaw)
		ui_set(stage_bodyyawval, 			curdata.bodyyawval)
		ui_set(stage_freestandingbodyyaw, 	curdata.freestandbodyyaw)
		ui_set(stage_lbytarget, 			curdata.lbytarget)
		ui_set(stage_fakelimit, 			curdata.fakelimit)
	end

	if cur > init_size and main_table[cur] ~= nil then 
		ui_set(stage_name, main_table[cur]) 
	end

	if cur == 1 then
		ui_set(stage_name, "")
	end

	ui_set_visible(main_listbox,				main)

	ui_set_visible(stage_add, 					main and cur == 1)

	ui_set_visible(config_listbox, 				main and cur == 2)
	ui_set_visible(config_name, 				main and cur == 2)
	ui_set_visible(config_save, 				main and cur == 2)
	ui_set_visible(config_load, 				main and cur == 2)
	ui_set_visible(config_remove, 				main and cur == 2)
	ui_set_visible(config_import, 				main and cur == 2)
	ui_set_visible(config_export, 				main and cur == 2)

	ui_set_visible(ind_selector,				main and cur == 3)
	ui_set_visible(ind_selector_color,			main and cur == 3 and ui_get(ind_selector) ~= "Off")

	ui_set_visible(stage_name, 	 				(main and cur == 1) or bool_stage)
	ui_set_visible(stage_duplicate, 			bool_stage)
	ui_set_visible(stage_remove, 				bool_stage)
	ui_set_visible(stage_moveup, 				bool_stage and cur > init_size+1)
	ui_set_visible(stage_movedown, 				bool_stage and cur < #main_table)

	ui_set_visible(stage_switch, 				bool_stage)
	ui_set_visible(stage_pitch,  				bool_stage)
	ui_set_visible(stage_yawbase,    			bool_stage)
	ui_set_visible(stage_yaw,    				bool_stage)
	ui_set_visible(stage_yawval, 				bool_stage and ui_get(stage_yaw) ~= "Off")
	ui_set_visible(stage_yawjitter,     		bool_stage)
	ui_set_visible(stage_yawjitterval,  		bool_stage and ui_get(stage_yawjitter) ~= "Off")
	ui_set_visible(stage_bodyyaw,				bool_stage)
	ui_set_visible(stage_bodyyawval,			bool_stage and ui_get(stage_bodyyaw) ~= "Off" and ui_get(stage_bodyyaw) ~= "Opposite")
	ui_set_visible(stage_freestandingbodyyaw,	bool_stage and ui_get(stage_bodyyaw) ~= "Off")
	ui_set_visible(stage_lbytarget, 			bool_stage and ui_get(stage_bodyyaw) ~= "Off")
	ui_set_visible(stage_fakelimit, 			bool_stage and ui_get(stage_bodyyaw) ~= "Off")
	ui_set_visible(stage_edgeyaw,    			bool_stage)
	ui_set_visible(stage_freestanding,    		bool_stage)
	ui_set_visible(stage_freestanding_hotkey,   bool_stage)

	ui_set_visible(stage_timer, 				bool_stage and ui_get(stage_switch) == "Timer")
	ui_set_visible(stage_hotkey, 				bool_stage and ui_get(stage_switch) == "Hotkey")
end

local function handle_stage_names()
	if client_key_state(0x0D) and (ui_get(main_listbox)+1 > init_size) and (main_table[ui_get(main_listbox)+1].name ~= ui_get(stage_name)) then
		main_table[ui_get(main_listbox)+1] = ui_get(stage_name)
		ui_update(main_listbox, main_table)
	end
end

local function handle_configlist()
	if #configs > 0 then
		ui_set(config_name, configs[ui_get(config_listbox)+1])
	end
end

local function handle_antiaim()
	local cur = ui_get(main_listbox)+1

	if #main_table <= init_size then return end

	if current_stage > #main_data then
		current_stage = init_size + 1
	end

	local curdata = main_data[current_stage]

	if curdata.switchtype == "Timer" then
		if timer == true then
			timer_timer = globals_realtime() + (curdata.timer*0.001) -- multiplication faster than division
			client_delay_call(0.0001, function() timer = false end)
		end

		if timer == false and timer_timer < globals_realtime() then
			timer = true
			should_switch = true
		end
	elseif curdata.switchtype == "Hotkey" then
		if ui_get(stage_hotkey) and can_switch then
			should_switch = true
			can_switch = false
		end

		if not ui_get(stage_hotkey) and not can_switch then
			can_switch = true
		end
	elseif curdata.switchtype == "Bruteforce" then
		if brute_timer < globals_curtime() then
			current_stage = init_size
			should_switch = true
			brute_timer = globals_curtime() + 99999 -- prevent it from reseting multiple times
		end
	end

	if ui_is_menu_open() and cur > init_size then
		current_stage = ui_get(main_listbox)
		should_switch = true
		can_switch_menu = true
	end

	if not ui_is_menu_open() and can_switch_menu then
		current_stage = init_size
		should_switch = true
		can_switch_menu = false
	end

	if should_switch then
		current_stage = current_stage + 1

		if current_stage > #main_data then
			current_stage = init_size + 1
		end

		curdata = main_data[current_stage]

		ui_set(references.pitch, curdata.pitch)
		ui_set(references.yawbase, ui_get(stage_yawbase))
		ui_set(references.yaw[1], curdata.yaw)
		ui_set(references.yaw[2], curdata.yawval)
		ui_set(references.yawjitter[1], curdata.yawjitter)
		ui_set(references.yawjitter[2], curdata.yawjitterval)
		ui_set(references.bodyyaw[1], curdata.bodyyaw)
		ui_set(references.bodyyaw[2], curdata.bodyyawval)
		ui_set(references.freestandbodyyaw, curdata.freestandbodyyaw)
		ui_set(references.lbytarget, curdata.lbytarget)
		ui_set(references.fakelimit, curdata.fakelimit)
		ui_set(references.edgeyaw, ui_get(stage_edgeyaw))
		ui_set(references.freestanding[1], ui_get(stage_freestanding))
		ui_set(references.freestanding[2], ui_get(stage_freestanding_hotkey) and "Always on" or "On hotkey")

		should_switch = false
	end
end

local function handle_indicators()
	if ui_get(ind_selector) == "Off" or #main_data == init_size then
		return
	end

	local r, g, b, a = ui_get(ind_selector_color)

	if ui_get(ind_selector) == "Indicator" then
		renderer_indicator(r, g, b, a, "["..current_stage-init_size.."] "..main_table[current_stage])
	elseif ui_get(ind_selector) == "Array" then
		for i = init_size+1, #main_table do
			local tw, th = renderer_measure_text("bd+", main_table[i])
			local space = th * 1.1

			renderer_gradient(sw - 10 - math_floor(tw * 0.33 + 0.5), sh * 0.67 - space * #main_table + space * i, tw * 0.33, th, 0, 0, 0, 75 * a * 0.004, 0, 0, 0, 0, true) -- multiplication faster than division
			renderer_gradient(sw - 10 - mathfloor(tw * 0.66), sh * 0.67 - space * #main_table + space * i, tw * 0.33, th, 0, 0, 0, 0, 0, 0, 0, 75 * a * 0.004, true)

			if i == current_stage then
				renderer_text(sw - 10, sh * 0.67 - space * #main_table + space * i, r, g, b, a, "brd+", 0, main_table[i])
			else
				renderer_text(sw - 10, sh * 0.67 - space * #main_table + space * i, r * 0.75, g * 0.75, b * 0.75, a * 0.55, "brd+", 0, main_table[i])
			end
		end
	end
end

local function handle_callbacks()
	local callback = ui_get(main_master_switch) and client_set_event_callback or client_unset_event_callback

	callback("weapon_fire", on_weapon_fire)
	callback("round_prestart", on_prestart)
	callback("pre_render", handle_stage_names)
	callback("bullet_impact", on_bullet_impact)
	callback("setup_command", handle_antiaim)
	callback("paint", handle_indicators)

	handle_visibility()
end
-- #endregion

-- #region ui callbacks
local function data_callback(ui_element, data)
	ui_set_callback(ui_element, function()
		if #main_data > init_size and main_data[ui_get(main_listbox)+1][data] ~= nil then
			main_data[ui_get(main_listbox)+1][data] = ui_get(ui_element)
		end
		handle_visibility()
	end)
end

data_callback(stage_switch, "switchtype")
data_callback(stage_timer, "timer")
data_callback(stage_pitch, "pitch")
data_callback(stage_yaw, "yaw")
data_callback(stage_yawval, "yawval")
data_callback(stage_yawjitter, "yawjitter")
data_callback(stage_yawjitterval, "yawjitterval")
data_callback(stage_bodyyaw, "bodyyaw")
data_callback(stage_bodyyawval, "bodyyawval")
data_callback(stage_freestandingbodyyaw, "freestandbodyyaw")
data_callback(stage_lbytarget, "lbytarget")
data_callback(stage_fakelimit, "fakelimit")

ui_set_callback(main_master_switch, 		handle_callbacks)
ui_set_callback(main_listbox, 				handle_visibility)
ui_set_callback(stage_yawbase, 				handle_visibility)
ui_set_callback(stage_edgeyaw, 				handle_visibility)
ui_set_callback(stage_freestanding, 		handle_visibility)
ui_set_callback(stage_freestanding_hotkey, 	handle_visibility)
ui_set_callback(config_listbox, 			handle_configlist)

ui_set_callback(stage_add, function() 
	main_table[#main_table+1] = ui_get(stage_name) ~= "" and ui_get(stage_name) or "Default"
	main_data[#main_data+1] = {
		switchtype = "Timer",
		timer = 100,
		pitch = "Default",
		yaw = "180",
		yawval = 0,
		yawjitter = "Off",
		yawjitterval = 0,
		bodyyaw = "Static",
		bodyyawval = 60,
		freestandbodyyaw = false,
		lbytarget = "Eye yaw",
		fakelimit = 60
	}

	ui_update(main_listbox, main_table)
end)

ui_set_callback(stage_duplicate, function() 
	main_table[#main_table+1] = ui_get(stage_name)
	main_data[#main_data+1] = copy_stage_data(main_data[ui_get(main_listbox)+1])

	handle_visibility()
	ui_update(main_listbox, main_table)
end)

ui_set_callback(stage_remove, function() 
	table_remove(main_table, ui_get(main_listbox)+1)
	table_remove(main_data, ui_get(main_listbox)+1)

	should_switch = true

	handle_visibility()
	ui_update(main_listbox, main_table)
end)

ui_set_callback(stage_moveup, function()
	local cur = ui_get(main_listbox)+1
	local temp = {}

	temp_name = main_table[cur]
	temp_data = copy_stage_data(main_data[cur])

	main_table[cur] = main_table[cur-1]
	main_data[cur] = main_data[cur-1]
	main_table[cur-1] = temp_name
	main_data[cur-1] = temp_data

	handle_visibility()
	ui_update(main_listbox, main_table)
end)

ui_set_callback(stage_movedown, function()
	local cur = ui_get(main_listbox)+1
	local temp = {}

	temp_name = main_table[cur]
	temp_data = copy_stage_data(main_data[cur])

	main_table[cur] = main_table[cur+1]
	main_data[cur] = main_data[cur+1]
	main_table[cur+1] = temp_name
	main_data[cur+1] = temp_data

	handle_visibility()
	ui_update(main_listbox, main_table)
end)

ui_set_callback(config_save, function() 
	if #main_table == init_size or ui_get(config_name) == "" then 
		return
	end

	local config_exists = false

	local temp = {}
	for i = init_size+1, #main_data do
		temp[#temp+1] = {
			id = main_table[i],
			data = copy_stage_data(main_data[i]),
			extras = {
				yawbase = ui_get(stage_yawbase),
				edgeyaw = ui_get(stage_edgeyaw),
				freestanding = ui_get(stage_freestanding)
			}
		}
	end

	for _, v in pairs(configs) do
		if v == ui_get(config_name) then
			config_exists = true
		end
	end

	if not config_exists then
		configs[#configs+1] = ui_get(config_name)
	end

	database_write("aabuilder_configs_"..ui_get(config_name), temp)
	handle_visibility()
	ui_update(config_listbox, configs)
end)

ui_set_callback(config_load, function() 
	local data = database_read("aabuilder_configs_"..ui_get(config_name))

	if data == nil then
		return
	end

	clear_stage_data()
	for i = 1, #data do
		main_table[#main_table+1] = data[i].id
		main_data[#main_data+1] = data[i].data
	end

	ui_set(stage_yawbase, data[1].extras.yawbase)
	ui_set(stage_edgeyaw, data[1].extras.edgeyaw)
	ui_set(stage_freestanding, data[1].extras.freestanding)

	handle_visibility()
	ui_update(main_listbox, main_table)
	ui_update(config_listbox, configs)
end)

ui_set_callback(config_remove, function()
	database_write("aabuilder_configs_"..configs[ui_get(config_listbox)+1], nil)
	table_remove(configs, ui_get(config_listbox)+1)
	handle_visibility()
	ui_update(config_listbox, configs)
end)

ui_set_callback(config_import, function() 
	if clipboard_get() == nil then
		return
	end

	local import_string = loadstring(clipboard.get())()

	clear_stage_data()

	for i = 1, #import_string[1] do
		main_table[#main_table+1] = import_string[1][i]
		main_data[#main_data+1] = import_string[2][i]
	end

	handle_visibility()
	ui_update(main_listbox, main_table)
end)

ui_set_callback(config_export, function() 
	if #main_table == init_size then
		return
	end

	local export_string = "return {\n{" -- Im the one that made this and idk how it works properly
	for i = init_size+1, #main_table do
		export_string = export_string.. "\"".. main_table[i].. "\","
	end
	export_string = export_string.. "},\n{"
	for i = init_size+1, #main_data do
		local table_to_copy = copy_stage_data(main_data[i])

		export_string = export_string.. "{"

		for k, v in pairs(main_data[i]) do
			if type(v) == "string" then
				export_string = export_string.. tostring(k).. " = \"".. tostring(v).. "\", "
			else
				export_string = export_string.. tostring(k).. " = ".. tostring(v).. ", "
			end
		end

		export_string = export_string.. "},"
	end
	export_string = export_string.. "}}"

	clipboard_set(export_string)
end)
-- #endregion

-- #region initialize
handle_callbacks()
handle_configlist()
client_delay_call(0.0001, handle_visibility)

client_set_event_callback("player_connect_full", on_connect_full)
client_set_event_callback("shutdown", on_shutdown)
-- #endregion
