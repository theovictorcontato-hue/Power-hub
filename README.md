-- Power X Hub | 99 Noites na Floresta (WindUI FIXED 2026) - Théo RJ
-- Fixes: Hunger/Saplings detecção, pcall safety, encoding clean, melhor auto chop

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Player = game.Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Update char on respawn
Player.CharacterAdded:Connect(function(newChar)
    Character = newChar
    Humanoid = newChar:WaitForChild("Humanoid")
    RootPart = newChar:WaitForChild("HumanoidRootPart")
end)

local Window = WindUI:CreateWindow({
    Name = "Power X Hub | 99 Noites na Floresta 🔥",
    LoadingTitle = "Carregando Hub...",
    LoadingSubtitle = "by Théo (Rio de Janeiro)",
    ConfigurationSaving = {Enabled = true, FolderName = "PowerXHub", FileName = "99NightsConfig"},
    KeySystem = false
})

local MainTab = Window:Tab({Title = "Principal", Icon = "home"})
local MainSection = MainTab:Section({Title = "Features Essenciais", Side = "Left"})

-- God Mode (mais robusto)
MainSection:Toggle({
    Title = "God Mode (Saúde Infinita)",
    Flag = "GodMode",
    Default = false,
    Callback = function(Value)
        if Value then
            pcall(function()
                Humanoid.MaxHealth = math.huge
                Humanoid.Health = math.huge
            end)
            spawn(function()
                while WindUI.Flags.GodMode and Humanoid do
                    pcall(function() Humanoid.Health = math.huge end)
                    wait(0.3)
                end
            end)
        else
            pcall(function()
                Humanoid.MaxHealth = 100
                Humanoid.Health = 100
            end)
        end
    end
})

-- Infinite Hunger (tentativa melhor: procura em PlayerGui ou assume regen)
MainSection:Toggle({
    Title = "Fome Infinita",
    Flag = "InfiniteHunger",
    Default = false,
    Callback = function(Value)
        spawn(function()
            while WindUI.Flags.InfiniteHunger do
                -- Tenta achar hunger bar (comum em PlayerGui > HungerBar ou similar)
                local hungerGui = Player:FindFirstChild("PlayerGui") and Player.PlayerGui:FindFirstChildWhichIsA("ScreenGui"):FindFirstChild("Hunger", true)
                if hungerGui and hungerGui:IsA("Frame") then
                    -- Alguns usam Size ou Value; tenta setar visual
                    pcall(function() hungerGui.Size = UDim2.new(1,0,0,20) end) -- full bar hack visual
                end
                -- Se for Value real (raro), set
                for _, v in pairs(Player:GetDescendants()) do
                    if v.Name:lower():find("hunger") and v:IsA("NumberValue") then
                        v.Value = 100
                    end
                end
                wait(1)
            end
        end)
    end
})

-- Infinite Saplings (melhor busca em inventory/leaderstats)
MainSection:Toggle({
    Title = "Mudas Infinitas",
    Flag = "InfiniteSaplings",
    Default = false,
    Callback = function(Value)
        spawn(function()
            while WindUI.Flags.InfiniteSaplings do
                for _, v in pairs(Player:GetDescendants()) do
                    if v.Name:lower():find("sapling") or v.Name:lower():find("muda") or v.Name == "Saplings" then
                        if v:IsA("IntValue") or v:IsA("NumberValue") then
                            v.Value = 9999
                        end
                    end
                end
                wait(2)
            end
        end)
    end
})

-- Kill Aura (com pcall)
local AuraRange = 15
MainSection:Slider({Title = "Alcance Kill Aura", Default = 15, Min = 5, Max = 50, Callback = function(v) AuraRange = v end})

MainSection:Toggle({
    Title = "Kill Aura (Auto Mata Mobs)",
    Flag = "KillAura",
    Default = false,
    Callback = function(Value)
        spawn(function()
            while WindUI.Flags.KillAura do
                pcall(function()
                    for _, v in pairs(workspace:GetDescendants()) do
                        if v:IsA("Model") and v:FindFirstChild("Humanoid") and v ~= Character and v.Humanoid.Health > 0 then
                            local hrp = v:FindFirstChild("HumanoidRootPart")
                            if hrp and (hrp.Position - RootPart.Position).Magnitude <= AuraRange then
                                v.Humanoid.Health = 0
                            end
                        end
                    end
                end)
                wait(0.15)
            end
        end)
    end
})

-- Auto Chop (busca melhor por tree models)
MainSection:Toggle({
    Title = "Auto Chop / Farm Madeira",
    Flag = "AutoChop",
    Default = false,
    Callback = function(Value)
        spawn(function()
            while WindUI.Flags.AutoChop do
                pcall(function()
                    for _, tree in pairs(workspace:GetChildren()) do
                        if tree:IsA("Model") and (tree.Name:lower():find("tree") or tree.Name:lower():find("oak") or tree.Name:lower():find("pine")) then
                            local part = tree:FindFirstChild("Trunk") or tree:FindFirstChild("Hitbox") or tree:FindFirstChildWhichIsA("BasePart")
                            if part then
                                firetouchinterest(RootPart, part, 0)
                                wait(0.05)
                                firetouchinterest(RootPart, part, 1)
                            end
                        end
                    end
                end)
                wait(0.6)
            end
        end)
    end
})

-- Bring Items (com filtro melhor)
MainSection:Toggle({
    Title = "Bring All Items",
    Flag = "BringItems",
    Default = false,
    Callback = function(Value)
        spawn(function()
            while WindUI.Flags.BringItems do
                pcall(function()
                    for _, item in pairs(workspace:GetDescendants()) do
                        if item:IsA("BasePart") and (item.Name:find("Diamond") or item.Name:find("Sapling") or item.Name:find("Chest") or item.Name:find("Log")) then
                            item.CFrame = RootPart.CFrame + Vector3.new(0, 4, 0)
                        end
                    end
                end)
                wait(0.4)
            end
        end)
    end
})

-- Teleports (já bons, adicionei mais robustez)
local TpTab = Window:Tab({Title = "Teleporte"})
local TpSection = TpTab:Section({Title = "Teleportes Rápidos"})

TpSection:Button({
    Title = "TP Fogueira / Campfire",
    Callback = function()
        local found = false
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj.Name:lower():find("campfire") or obj.Name:lower():find("fogueira") then
                if obj:IsA("BasePart") or obj:IsA("Model") and obj.PrimaryPart then
                    RootPart.CFrame = (obj.PrimaryPart or obj).CFrame + Vector3.new(0,5,0)
                    found = true
                    break
                end
            end
        end
        if not found then WindUI:Notify({Title = "TP", Content = "Fogueira não achada!", Duration = 3}) end
    end
})

-- Visual Tab (ESP limpo)
local VisualTab = Window:Tab({Title = "Visual"})
local VisualSection = VisualTab:Section({Title = "ESP"})

VisualSection:Toggle({
    Title = "Item ESP",
    Flag = "ItemESP",
    Default = false,
    Callback = function(Value)
        if Value then
            for _, part in pairs(workspace:GetDescendants()) do
                if part:IsA("BasePart") and (part.Name:find("Diamond") or part.Name:find("Sapling") or part.Name:find("Chest")) then
                    local hl = Instance.new("Highlight", part)
                    hl.FillColor = Color3.new(0,1,0)
                    hl.OutlineColor = Color3.new(1,1,1)
                    hl.FillTransparency = 0.4
                end
            end
        else
            for _, hl in pairs(workspace:GetDescendants()) do
                if hl:IsA("Highlight") then hl:Destroy() end
            end
        end
    end
})

Window:SetToggleKey(Enum.KeyCode.RightControl)

print("Power X Hub FIXED carregado! Testa God + Kill Aura primeiro 🔥")
WindUI:Notify({Title = "Loaded", Content = "Hub pronto! Se hunger não funcionar, avisa o nome exato da barra.", Duration = 5})
