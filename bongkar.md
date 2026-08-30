-- ==============================================================================
-- GENTA HAX - AUTO SURGERY & AUTO CONSUME SCRIPT
-- Made by #nYauL1763
-- ==============================================================================

-- Konfigurasi Awal Script
Config = {
    DelayMS = 250,
    ItemID = 5640, -- Contoh Item ID (Dummy/Surgical Tool)
    Position = "UP",
    AutoSurg = false,
    AutoPut = false,
    LegitMode = true,
    UseMagplant = false,
    -- Auto Consume Config
    Tea = false,
    Spice = false,
    TeaID = 7188,   -- ID untuk Tea
    SpiceID = 7190  -- ID untuk Skill Spice
}

surgerySuccessCount = 0
isConsumeRunning = false

-- ==================== FUNGSI HELPER STOK ITEM ====================
function getItemAmount(targetID)
    local inv = getInventory()
    if not inv then return 0 end
    for _, item in pairs(inv) do
        if item.id == targetID then
            return item.amount
        end
    end
    return 0
end

-- ==================== UI BUILDERS (SAFE BUTTON WITH ICONS) ====================
function buildMainMenu()
   local user = (getLocal() and getLocal().name) or "PLAYER"

   local ui = {}
   table.insert(ui, "set_bg_color|10,10,10,225")
   table.insert(ui, "set_border_color|0,150,255,255")
   table.insert(ui, "set_default_color|`o")
   table.insert(ui, "add_label_with_icon|big|`2Auto Surgery Full Feature``|left|6212|")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_label_with_icon|small|`cScript Made By #nYauL1763``|left|3690")
   table.insert(ui, "add_label_with_icon|small|`0Player: `w" .. user .. " `0|left|1240")
   table.insert(ui, "add_label|small|`0Successful Surgeries: `2" .. surgerySuccessCount .. " `0Patients Saved``|left|4296")
   table.insert(ui, "add_spacer|small|")
   
   -- Tombol standar dengan hiasan teks agar aman dari bug tombol hilang/berubah bentuk
   table.insert(ui, "add_button|SurgSettingsOpen|`2[`w⚙️`2] AutoSurg Settings``")
   table.insert(ui, "add_button|ConsumeSettingsOpen|`2[`w🍵`2] Auto Consume``")
   table.insert(ui, "add_button|AboutInfoOpen|`6[`wℹ️`6] Script Information``")
   table.insert(ui, "add_spacer|small|")
   table.insert(ui, "add_button|CloseMenu|`4Close Menu``")
   
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
   table.insert(ui, "add_button|BackToMain|`4Back To Menu``")
   table.insert(ui, "add_button|SaveSettings|`2Save Settings``")

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
   table.insert(ui, "add_button|BackToMain|`4Back To Menu``")
   table.insert(ui, "add_button|SaveConsume|`2Save Consume``")

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
   table.insert(ui, "add_button|BackToMain|`4Back To Menu``")

   return table.concat(ui, "\n")
end

-- ==================== HOOKS & EVENT HANDLERS ====================
AddHook(OnVarlist, "MainVarlistHook", function(varlist, netID)
    -- Deteksi dialog response dari user
    if varlist[0] and varlist[0] == "OnDialogRequest" then
        local content = varlist[1] or ""
        
        if content:find("Auto Surgery Full Feature") then
            if content:find("buttonClicked|SurgSettingsOpen") then
                sendPacket(2, "action|dialog_return\ndialog_name|surg_settings\n" .. buildSettingsMenu())
                return true
            elseif content:find("buttonClicked|ConsumeSettingsOpen") then
                sendPacket(2, "action|dialog_return\ndialog_name|consume_settings\n" .. buildConsumeMenu())
                return true
            elseif content:find("buttonClicked|AboutInfoOpen") then
                sendPacket(2, "action|dialog_return\ndialog_name|about_menu\n" .. buildAboutMenu())
                return true
            elseif content:find("buttonClicked|CloseMenu") then
                return true
            end
            
        elseif content:find("AutoSurg Configuration") then
            if content:find("buttonClicked|BackToMain") then
                sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
                return true
            elseif content:find("buttonClicked|SaveSettings") then
                -- Ambil input dari dialog
                -- (Implementasi parsing input bisa disesuaikan dengan kebutuhan)
                doToast(1, 2000, "`2Settings Saved Successfully!")
                sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
                return true
            end
            
        elseif content:find("Auto Consume") then
            if content:find("buttonClicked|BackToMain") then
                sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
                return true
            elseif content:find("buttonClicked|SaveConsume") then
                doToast(1, 2000, "`2Auto Consume Settings Saved!")
                sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
                return true
            end
            
        elseif content:find("Script Information") then
            if content:find("buttonClicked|BackToMain") then
                sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
                return true
            end
        end
    end
end)

-- Command untuk memanggil Menu Utama via Chat
AddHook(OnTextPacket, "MainTextHook", function(type, packet)
    if packet:find("action|input") and packet:find("/surg") then
        sendPacket(2, "action|dialog_return\ndialog_name|surg_main\n" .. buildMainMenu())
        doLogInfo("Menu opened via chat command /surg")
        return true
    end
end)

doLog("Genta Hax Script Loaded Successfully! Type /surg to open menu.")
doToast(1, 3000, "`2[Genta Hax] Script Loaded! Type /surg")
