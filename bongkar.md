--========================================================
-- Boolean Flags System v2.0 :crown:
-- Style panel pakai function dialog()
--========================================================

local booleanFlags = false
local isRunning = false

-- Fungsi delay aman
function sleep(ms)
    local start = os.time()
    repeat until os.difftime(os.time(), start) * 1000 >= ms
end

-- Fungsi kirim overlay
function send(txt)
    local var = {}
    var[0] = "OnTextOverlay"
    var[1] = "`3[👑 `6King Erik's BooleanFlags`3] `9" .. txt
    sendVariant(var)
end

-- Fungsi utama sistem
function mainSystem()
    if isRunning then return end
    isRunning = true
    send("`2Sistem dimulai... 👑")

    while booleanFlags do
        -- 🔁 Di sinilah sistem utama kamu berjalan
        send("`9Sistem berjalan otomatis...")
        sleep(2000)
    end

    send("`4Sistem dihentikan ❌")
    isRunning = false
end

-- Fungsi toggle booleanFlags
function toggleBooleanFlags()
    booleanFlags = not booleanFlags
    if booleanFlags then
        send("`2Sistem diaktifkan :crown:")
        runThread(mainSystem)
    else
        send("`4Sistem dimatikan ❌")
    end
end

-- Fungsi dialog dengan gaya seperti contohmu
function dialog()
    local thisDialog = [[
set_bg_color|10,15,35,235
set_border_color|255,215,0,200
add_label_with_icon|big|`eBooleanFlags System :crown:|left|3802|
add_spacer|small|
add_button_with_icon|startSys|`2Mulai Sistem Sekarang|staticBlueFrame|3802|
end_dialog|flagDialog|OK|
]]


    sendVariant({
        [0] = "OnDialogRequest",
        [1] = thisDialog
    })
end

-- Hook tombol OK ditekan
AddHook("OnDialogReturn", "BOOLEAN_FLAG_DIALOG", function(dialogID, fields)
    if dialogID == "flagDialog" and fields and fields.startSys then
        toggleBooleanFlags()
        -- Tutup panel otomatis
        sendVariant({[0] = "OnDialogRequest", [1] = ""})
    end
end)

-- Tampilkan panel
dialog()```
