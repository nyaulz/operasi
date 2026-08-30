-- Script Auto Surg Support Magplant & Genta Hax + LimitTea Logic (Clave Style UI + Blue Border + GT Native Buttons with Icons)
-- Remaked & Configured By #nYauL1763 | Type /surg To Start Script Menu

-- ==================== CONFIG & VARIABLES ====================
Config = {
   AutoSurg = true,
   AutoPut = false,
   LegitMode = false,
   UseMagplant = false,
   DelayMS = 1000,
   ItemID = 4296,
   Position = "RIGHT",
   
   -- Config Auto Consume (Genta Hax Style)
   Tea = false,
   Spice = false,
   TeaID = 5114,
   SpiceID = 6912,
   TeaDelay = 125,      -- 2 menit 5 detik (125 detik)
   SpiceDelay = 1805,   -- 30 menit 5 detik (1805 detik)
   Tick = 1
}

local tool = "0"
local isProcessing = false
local isConsumeRunning = false
local teaTimer, spiceTimer = 0, 0
local surgerySuccessCount = 0
local isSurgeryActive = false

local function logMsg(text)
   pcall(function()
      if Log then
         Log(text)
      elseif logToConsole then
         logToConsole(text)
      else
         print("[#nYauL1763-Surg] " .. tostring(text))
      end
   end)
end

function getItemAmount(itemID)
   local inv = getInventory()
   if not inv then return 0 end
   for _, item in pairs(inv) do
      if tonumber(item.id) == tonumber(itemID) then
         return tonumber(item.amount) or 0
      end
   end
   return 0
end

-- Tabel ID Alat Medis Original LIMITTEA
local toolIDs = {
    ["Sponge"]        = 1258,
    ["Scalpel"]       = 1260,
    ["Anesthetic"]    = 1262,
    ["Antibiotic"]    = 1266,
    ["Splint"]        = 1268,
    ["Stitches"]      = 1270,
    ["Fix it"]        = 1296,
    ["Pins"]          = 4308,
    ["Transfusion"]   = 4310,
    ["Defibrillator"] = 4312,
    ["Clamp"]         = 4314,
    ["Ultrasound"]    = 4316,
    ["Lab kit"]       = 4318
}

-- ==================== UI BUILDERS (WITH BUTTON ICONS) ====================
function buildMainMenu()
   local world = (getWorld() and getWorld().name) or "EXIT"
   local user = (getLocal() and getLocal().name) or "PLAYER"

   local ui = {}
   table.insert(ui, "set_bg_color|10,10,10,225")
   table.insert(ui, "set_border_color|0,150,255,255") -- Border Biru Muda
   table.insert(ui, "set_default_color|`o")
   table.insert(ui, "add_label_with_icon|big|`2Auto Surgery Full Feature``|left|6212|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label_with_icon|small|`cScript Made By #nYauL1763``|left|3690")
   table.insert(ui, "add_label_with_icon|small|`0Player: `w" .. user .. " `0|left|1240")
   table.insert(ui, "add_label|small|`0Successful Surgeries: `2" .. surgerySuccessCount .. " `0Patients Saved``|left|4296")
   table.insert(ui, "add_spacer|small|")
   
   -- Tombol dengan ikon menggunakan add_button_with_icon
   table.insert(ui, "add_button_with_icon|SurgSettingsOpen|`2AutoSurg Settings``|staticBlueFrame|4308|")
   table.insert(ui, "add_button_with_icon|ConsumeSettingsOpen|`2Auto Consume``|staticBlueFrame|5114|")
   table.insert(ui, "add_button_with_icon|AboutInfoOpen|`6Script Information``|staticBlueFrame|14162|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_button_with_icon|CloseMenu|`4Close Menu``|staticBlueFrame|32|")
   
   return table.concat(ui, "\n")
end

function buildSettingsMenu()
   local chkSurg = Config.AutoSurg and "1" or "0"
   local chkPut = Config.AutoPut and "1" or "0"
   local chkLegit = Config.LegitMode and "1" or "0"
   local chkMag = Config.UseMagplant and "1" or "0"

   local ui = {}
   table.insert(ui, "set_bg_color|10,10,10,225")
   table.insert(ui, "set_border_color|0,150,255,255")
   table.insert(ui, "set_default_color|`o")
   table.insert(ui, "add_label_with_icon|big|`eAutoSurg Configuration``|left|4308|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label|small|`9Configure your surgery automation settings below:``|left")
   table.insert(ui, "add_spacer|small|")
   
   table.insert(ui, "add_text_input|delay_ms|`wDelay Using Tools (ms):|" .. tostring(Config.DelayMS) .. "|4|")
   table.insert(ui, "add_text_input|item_id|`wCustom Item ID (Dummy):|" .. tostring(Config.ItemID) .. "|4|")
   table.insert(ui, "add_text_input|put_pos|`wPosition Place(UP/MID/LEFT/RIGHT):|" .. Config.Position .. "|5|")
   table.insert(ui, "add_spacer|small|")

   table.insert(ui, "add_checkbox|enable_surg|`0Enable Auto Surgery|" .. chkSurg)
   table.insert(ui, "add_checkbox|enable_put|`0Enable Auto Put / Place Item|" .. chkPut)
   table.insert(ui, "add_checkbox|enable_legit|`0Enable Legit Helper Mode|" .. chkLegit)
   table.insert(ui, "add_checkbox|enable_mag|`0Enable Using Magplant Remote|" .. chkMag)
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_button_with_icon|BackToMain|`4Back To Menu``|staticBlueFrame|32|")
   table.insert(ui, "add_button_with_icon|SaveSettings|`2Save Settings``|staticBlueFrame|6342|")

   return table.concat(ui, "\n")
end

function buildConsumeMenu()
   local chkTea = Config.Tea and "1" or "0"
   local chkSpice = Config.Spice and "1" or "0"
   local consumeStatus = isConsumeRunning and "`2[ACTIVE]``" or "`4[INACTIVE]``"

   local ui = {}
   table.insert(ui, "set_bg_color|10,10,10,225")
   table.insert(ui, "set_border_color|0,150,255,255")
   table.insert(ui, "set_default_color|`o")
   table.insert(ui, "add_label_with_icon|big|`eAuto Consume " .. consumeStatus .. "``|left|5114|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label|small|`9Automatic consumption for boosters:``|left")
   table.insert(ui, "add_spacer|small|")

   table.insert(ui, "add_checkbox|cb_tea|`0Tea (2m) [Stock: `e" .. getItemAmount(Config.TeaID) .. "`0]|" .. chkTea)
   table.insert(ui, "add_checkbox|cb_spice|`0Skill Spice (30m) [Stock: `e" .. getItemAmount(Config.SpiceID) .. "`0]|" .. chkSpice)
   table.insert(ui, "add_text_input|tea_id|`wTea Item ID:|" .. tostring(Config.TeaID) .. "|5|")
   table.insert(ui, "add_text_input|spice_id|`wSkill Spice Item ID:|" .. tostring(Config.SpiceID) .. "|5|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_button_with_icon|BackToMain|`4Back To Menu``|staticBlueFrame|32|")
   table.insert(ui, "add_button_with_icon|SaveConsume|`2Save Consume``|staticBlueFrame|6342|")

   return table.concat(ui, "\n")
end

function buildAboutMenu()
   local ui = {}
   table.insert(ui, "set_bg_color|10,10,10,225")
   table.insert(ui, "set_border_color|0,150,255,255")
   table.insert(ui, "set_default_color|`o")
   table.insert(ui, "add_label_with_icon|big|`6Script Information``|left|14162|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label_with_icon|small|`wScript Made By: `6#nYauL1763``|left|1240|")
   table.insert(ui, "add_label_with_icon|small|`wUI Style: `eblue Border + GT Native Buttons``|left|660|")
   table.insert(ui, "add_label_with_icon|small|`wFeatures Added: `6Legit Helper & Auto Consume``|left|32|")
   table.insert(ui, "add_spacer|medium|")
   table.insert(ui, "add_button_with_icon|BackToMain|`4Back To Menu``|staticBlueFrame|32|")

   return table.concat(ui, "\n")
end

local function showLegitHelper(toolName, surgeryDialog)
   local toolID = toolIDs[toolName]
   local combinedDialog = surgeryDialog

   if not combinedDialog:find("set_border_color") then
      combinedDialog = "set_bg_color|10,10,10,225\nset_border_color|0,150,255,255\n" .. combinedDialog
   else
      combinedDialog = combinedDialog:gsub("set_border_color|[%d,]+", "set_border_color|0,150,255,255")
      combinedDialog = combinedDialog:gsub("set_bg_color|[%d,]+", "set_bg_color|10,10,10,225")
   end

   if toolID then
      if combinedDialog:find("tool" .. toolID) then
         combinedDialog = combinedDialog:gsub("(%w+|tool" .. toolID .. "|)([^|\n]+)", function(prefix, label)
            return prefix .. "`e" .. toolName
         end, 1)
      end
   end

   if sendVariant then
      sendVariant({
         [0] = "OnDialogRequest",
         [1] = combinedDialog
      })
   end

   logMsg("`6[`6#nYauL1763 Helper Surg`6] `eRecommended: " .. tostring(toolName))
end

function showMenu(menuType)
   local content = ""
   if menuType == "main" then content = buildMainMenu()
   elseif menuType == "settings" then content = buildSettingsMenu()
   elseif menuType == "consume" then content = buildConsumeMenu()
   elseif menuType == "about" then content = buildAboutMenu()
   end

   sendVariant({[0] = "OnDialogRequest", [1] = content})
end

-- ==================== AUTO CONSUME LOGIC ====================
function consumeItem(itemID)
   local count = getItemAmount(itemID)
   if count <= 0 then return false end

   local player = getLocal()
   if player then
      local tileX = math.floor(player.pos.x / 32)
      local tileY = math.floor(player.pos.y / 32)
      local success = pcall(function()
         requestTileChange(tileX, tileY, itemID)
      end)
      return success
   end
   return false
end

function consumeLoop()
   if Config.Tea then consumeItem(Config.TeaID) end
   if Config.Spice then 
      sleep(1200) 
      consumeItem(Config.SpiceID) 
   end

   teaTimer, spiceTimer = 0, 0

   while isConsumeRunning do
      sleep(Config.Tick * 1000)
      if not isConsumeRunning then break end

      teaTimer = teaTimer + Config.Tick
      spiceTimer = spiceTimer + Config.Tick

      if Config.Tea and teaTimer >= Config.TeaDelay then
         consumeItem(Config.TeaID)
         teaTimer = 0
      end
      if Config.Spice and spiceTimer >= Config.SpiceDelay then
         consumeItem(Config.SpiceID)
         spiceTimer = 0
      end
   end
end

-- ==================== SURGERY AUTOMATION ====================
function auto()
    local itool = toolIDs[tool]
    if not itool then return end
    
    sendPacket(2, "action|dialog_return\ndialog_name|surgery\nbuttonClicked|tool" .. itool)
    logMsg("`6[`6#nYauL1763 Tools`6] `c" .. tool)
end

function afterSurg()
   sleep(1000)
   isSurgeryActive = false
   if not Config.AutoPut or not Config.AutoSurg then return end

   local checkID = Config.UseMagplant and 5640 or Config.ItemID
   local count = getItemAmount(checkID)
   if count <= 0 then
      logMsg("`6[`6#nYauL1763 AutoSurg`6] `4Item ID " .. checkID .. " Empty! Auto Put Deactivated.")
      Config.AutoPut = false
      return
   end

   local localPlayer = getLocal()
   if not localPlayer then return end

   local px = math.floor(localPlayer.pos.x / 32)
   local py = math.floor(localPlayer.pos.y / 32)

   local targetX, targetY = px, py
   local posUpper = Config.Position:upper()

   if posUpper == "UP" then targetY = py - 1
   elseif posUpper == "LEFT" then targetX = px - 1
   elseif posUpper == "RIGHT" then targetX = px + 1
   elseif posUpper == "MID" then targetX, targetY = px, py
   else targetX = px + 1 end

   requestTileChange(targetX, targetY, checkID)
   sleep(1500)

   sendPacketRaw(false, {
      type = 3,
      value = 32,
      x = localPlayer.pos.x,
      y = localPlayer.pos.y,
      punchx = targetX,
      punchy = targetY
   })
end

-- ==================== HOOKS ====================
AddHook("OnTextPacket", "SurgCmdHook", function(type_pkt, pkt)
   if not pkt then return false end

   if pkt:find("/surg") or pkt:find("/consume") then
      showMenu("main")
      return true
   end

   if pkt:find("buttonClicked|SurgSettingsOpen") then
      showMenu("settings")
      return true
   elseif pkt:find("buttonClicked|ConsumeSettingsOpen") then
      showMenu("consume")
      return true
   elseif pkt:find("buttonClicked|AboutInfoOpen") then
      showMenu("about")
      return true
   elseif pkt:find("buttonClicked|BackToMain") then
      showMenu("main")
      return true
   elseif pkt:find("buttonClicked|CloseMenu") then
      return true
   end

   if pkt:find("buttonClicked|SaveSettings") then
      Config.AutoSurg = (pkt:match("enable_surg|(%d+)") == "1")
      Config.AutoPut = (pkt:match("enable_put|(%d+)") == "1")
      Config.LegitMode = (pkt:match("enable_legit|(%d+)") == "1")
      Config.UseMagplant = (pkt:match("enable_mag|(%d+)") == "1")

      Config.DelayMS = tonumber(pkt:match("delay_ms|(%d+)")) or Config.DelayMS
      Config.ItemID = tonumber(pkt:match("item_id|(%d+)")) or Config.ItemID
      Config.Position = tostring(pkt:match("put_pos|(%a+)") or Config.Position):upper()

      logMsg("`6[Auto Surg Settings] `2Settings Saved Successfully!")
      showMenu("main")
      return true
   end

   if pkt:find("buttonClicked|SaveConsume") then
      Config.Tea = (pkt:match("cb_tea|(%d+)") == "1")
      Config.Spice = (pkt:match("cb_spice|(%d+)") == "1")
      Config.TeaID = tonumber(pkt:match("tea_id|(%d+)")) or Config.TeaID
      Config.SpiceID = tonumber(pkt:match("spice_id|(%d+)")) or Config.SpiceID

      logMsg("`6[Auto Consume] `2Consume Settings Saved!")

      if Config.Tea or Config.Spice then
         if not isConsumeRunning then
            isConsumeRunning = true
            runThread(consumeLoop)
         else
            isConsumeRunning = false
            sleep(300)
            isConsumeRunning = true
            runThread(consumeLoop)
         end
      else
         isConsumeRunning = false
      end

      showMenu("main")
      return true
   end

   return false
end)

AddHook("OnVarlist", "SurgMainHook", function(var)
   if not var then return false end

   local v0 = tostring(var[0] or var.v0 or "")
   local v1 = tostring(var[1] or var.v1 or "")

   if v0 == "OnConsoleMessage" and v1:find("YOU SAVED YOUR PATIENT!") then
      surgerySuccessCount = surgerySuccessCount + 1
      logMsg("`6Successful Surgeries: `2" + surgerySuccessCount)
      if Config.AutoSurg then 
         runThread(afterSurg) 
      end
      return false
   end

   if Config.AutoSurg and not Config.LegitMode and v0 == "OnDialogRequest" and (v1:find("Anatomical Dummy") or v1:find("dialog_name|surge")) then
      local tilex = v1:match("embed_data|tilex|(%d+)")
      local tiley = v1:match("embed_data|tiley|(%d+)")

      if tilex and tiley then
         runThread(function()
            sleep(300)
            sendPacket(2, "action|dialog_return\ndialog_name|surge\ntilex|" .. tilex .. "\ntiley|" .. tiley .. "\nbuttonClicked|okay")
         end)
         return true
      end
   end

   if not Config.AutoSurg then return false end
   if not (v0 == "OnDialogRequest" and (v1:find("surgery") or v1:find("tool1260"))) then return false end
   
   if not isSurgeryActive then
      isSurgeryActive = true
      logMsg("`6[`6#nYauL1763 AutoSurg`6] `2Ready Surg...")
   end

   if isProcessing then return false end

   if v1:find("`4The patient wakes up!") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("`4The patient screams and flails!") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Status: `4Heart stopped!(.+)") and v1:find("tool4312") then tool = "Defibrillator"
   elseif v1:find("Status: `6Coming to(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Pulse: `4(.+)") and v1:find("tool4310") then tool = "Transfusion"
   elseif v1:find("Temp: `4(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `4(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Temp: `6(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `6(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Temp: `3(%d+)(.+)") and v1:find("tool1266") then tool = "Antibiotic"
   elseif v1:find("Temp: `3(%d+)(.+)") and v1:find("tool4318") then tool = "Lab kit"
   elseif v1:find("Patient is losing blood `4very quickly!(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("Patient is losing blood `4very quickly!(.+)") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Patient is `6losing blood!(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("Patient is `6losing blood!(.+)") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Incisions: `20(.+)") and v1:find("tool1296") then tool = "Fix it"
   elseif v1:find("Incisions: `30(.+)") and v1:find("tool1296") then tool = "Fix it"
   elseif v1:find("The patient has not been diagnosed.") and v1:find("tool4316") then tool = "Ultrasound"
   elseif v1:find("Status: `4Awake(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Bones: `6(.+) broken``") and v1:find("tool1268") then tool = "Splint"
   elseif v1:find("Bones: `4(.+) broken``") and v1:find("tool1268") then tool = "Splint"
   elseif v1:find("Patient broke his arm.") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Status: `3Awake(.+)") and v1:find("tool1262") then tool = "Anesthetic"
   elseif v1:find("Pulse: `6(.+)") and v1:find("tool4310") then tool = "Transfusion"
   elseif v1:find("`4You can't see what you are doing!(.+)") and v1:find("tool1258") then tool = "Sponge"
   elseif v1:find("tool1296") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Bones: `6(.+), `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+), `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+), `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+), `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `6(.+), `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+), `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+), `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+), `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `6(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `6(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Bones: `4(.+) shattered(.+)") and v1:find("tool4308") then tool = "Pins"
   elseif v1:find("Bones: `4(.+) shattered(.+)") and v1:find("tool1260") then tool = "Scalpel"
   elseif v1:find("Patient broke his leg.") and v1:find("tool1270") then tool = "Stitches"
   elseif v1:find("Patient is losing blood `3slowly.(.+)") and v1:find("tool4314") then tool = "Clamp"
   elseif v1:find("tool1260") then tool = "Scalpel"
   else return false end

   if Config.LegitMode then
      showLegitHelper(tool, v1)
      return true
   end

   isProcessing = true
   runThread(function()
      sleep(Config.DelayMS)
      auto()
      sleep(100)
      isProcessing = false
   end)
   return true
end)

logMsg("`6[`6#nYauL1763 SYSTEM`6] `aAuto Surgery Script Ready! Type /surg")
showMenu("main")
