-- LocalScript para Roblox (Luau)
-- Coloque este LocalScript em StarterGui ou StarterPlayerScripts (ele cria o ScreenGui no PlayerGui).
-- UI menor, título centralizado "LAVA CRASH", toggle on/off (verde/vermelho),
-- texto inferior com cores mais suaves, arrastável, desliga automaticamente 3s após ativado.
-- Notificação topo central: "LAVA CRASH crashando" (aparece ao ligar e desaparece após 2s)

local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Helper: converte hex para Color3
local function hexToColor3(hex)
	hex = hex:gsub("#","")
	local r = tonumber(hex:sub(1,2), 16) or 0
	local g = tonumber(hex:sub(3,4), 16) or 0
	local b = tonumber(hex:sub(5,6), 16) or 0
	return Color3.fromRGB(r,g,b)
end

-- Cria ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LavaCrashGUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
-- garante que fique acima de outras GUIs
screenGui.DisplayOrder = 100
screenGui.Parent = playerGui

-- ---------- NOTIFICAÇÃO NO TOPO CENTRAL (criada cedo) ----------
local notification = Instance.new("Frame")
notification.Name = "TopNotification"
notification.Parent = screenGui
notification.Size = UDim2.new(0, 360, 0, 44)
notification.AnchorPoint = Vector2.new(0.5, 0)
notification.Position = UDim2.new(0.5, 0, 0, -60) -- começa fora da tela (para animar descendo)
notification.BackgroundColor3 = Color3.fromRGB(26, 26, 26)
notification.BorderSizePixel = 0
notification.ZIndex = 50
notification.Visible = false

local notifCorner = Instance.new("UICorner")
notifCorner.CornerRadius = UDim.new(0, 12)
notifCorner.Parent = notification

local notifStroke = Instance.new("UIStroke")
notifStroke.Thickness = 1
notifStroke.Color = Color3.fromRGB(60, 60, 60)
notifStroke.Parent = notification

local notifLabel = Instance.new("TextLabel")
notifLabel.Name = "NotifLabel"
notifLabel.Size = UDim2.new(1, -20, 1, 0)
notifLabel.Position = UDim2.new(0, 10, 0, 0)
notifLabel.BackgroundTransparency = 1
notifLabel.Text = "LAVA CRASH crashando" -- garante texto
notifLabel.Font = Enum.Font.GothamBold
notifLabel.TextSize = 18
notifLabel.TextColor3 = Color3.fromRGB(255, 140, 80)
notifLabel.TextXAlignment = Enum.TextXAlignment.Center
notifLabel.TextYAlignment = Enum.TextYAlignment.Center
notifLabel.ZIndex = 51
notifLabel.Parent = notification

local notifTweenInfo = TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

-- token para cancelar autoclose da notificação
local notifAutoHideToken = nil

local function showNotification()
	-- cancela timer antigo (apenas sobrescrevendo o token)
	local token = {}
	notifAutoHideToken = token

	notification.Visible = true
	-- garante estado inicial para animação
	notification.Position = UDim2.new(0.5, 0, 0, -60)
	notification.BackgroundTransparency = 1
	notifLabel.TextTransparency = 1

	-- anima entrada
	TweenService:Create(notification, notifTweenInfo, {Position = UDim2.new(0.5, 0, 0, 12), BackgroundTransparency = 0}):Play()
	TweenService:Create(notifLabel, TweenInfo.new(0.18), {TextTransparency = 0}):Play()

	-- agenda esconder após 2 segundos (só se token ainda válido)
	task.delay(2, function()
		if notifAutoHideToken == token then
			-- chama a função de esconder (que também vai animar)
			local ok, err = pcall(function()
				-- in-case hideNotification throws, avoid hard error
				-- hideNotification está abaixo; chamamos diretamente
				-- Para evitar cycles, chamamos a função local abaixo
				-- (see hideNotification definition)
			end)
			-- usar a função real
			local suc, em = pcall(function() hideNotification() end)
			-- se por algum motivo hideNotification não existir, apenas ignora
		end
	end)
end

local function hideNotification()
	-- cancela token para evitar múltiplas chamadas futuras
	notifAutoHideToken = nil
	-- anima saída e depois esconde
	local t1 = TweenService:Create(notification, TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.In), {Position = UDim2.new(0.5, 0, 0, -60), BackgroundTransparency = 1})
	local t2 = TweenService:Create(notifLabel, TweenInfo.new(0.12), {TextTransparency = 1})
	t1:Play(); t2:Play()
	t1.Completed:Connect(function()
		-- garantir que não reabra se outro token tiver aparecido
		if notifAutoHideToken == nil then
			notification.Visible = false
		end
	end)
end

-- Frame principal (menor)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 320, 0, 160)
mainFrame.Position = UDim2.new(0.5, -160, 0.35, -80)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(34, 34, 36)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 16)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Thickness = 2
mainStroke.Color = Color3.fromRGB(60, 60, 60)
mainStroke.Parent = mainFrame

-- Título "LAVA CRASH" centralizado
local title = Instance.new("TextLabel")
title.Name = "Title"
title.Parent = mainFrame
title.Size = UDim2.new(1, -24, 0, 40)
title.Position = UDim2.new(0, 12, 0, 8)
title.BackgroundTransparency = 1
title.Text = "LAVA CRASH"
title.Font = Enum.Font.GothamBlack
title.TextSize = 20
title.TextColor3 = Color3.fromRGB(255, 140, 80)
title.TextXAlignment = Enum.TextXAlignment.Center
title.TextYAlignment = Enum.TextYAlignment.Center

-- Linha fina abaixo do título
local divider = Instance.new("Frame")
divider.Name = "Divider"
divider.Parent = mainFrame
divider.Size = UDim2.new(1, -24, 0, 1)
divider.Position = UDim2.new(0, 12, 0, 50)
divider.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
divider.BorderSizePixel = 0
divider.ZIndex = 2

-- Botão de toggle (centro)
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Parent = mainFrame
toggleButton.Size = UDim2.new(0.5, 0, 0, 44)
toggleButton.Position = UDim2.new(0.5, 0, 0, 72)
toggleButton.AnchorPoint = Vector2.new(0.5, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50) -- inicial vermelho (off)
toggleButton.BorderSizePixel = 0
toggleButton.Text = "off"
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 20
toggleButton.TextColor3 = Color3.fromRGB(255,255,255)
toggleButton.AutoButtonColor = false

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 12)
toggleCorner.Parent = toggleButton

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Thickness = 2
toggleStroke.Color = Color3.fromRGB(30,30,30)
toggleStroke.Parent = toggleButton

-- Texto inferior com cores menos saturadas
local footer = Instance.new("TextLabel")
footer.Name = "Footer"
footer.Parent = mainFrame
footer.Size = UDim2.new(1, -24, 0, 22)
footer.Position = UDim2.new(0, 12, 1, -30)
footer.BackgroundTransparency = 1
footer.TextXAlignment = Enum.TextXAlignment.Center
footer.TextYAlignment = Enum.TextYAlignment.Center
footer.Font = Enum.Font.Gotham
footer.TextSize = 13
footer.RichText = true
footer.TextWrapped = true

local function colorizePerChar(text, palette)
	local parts = {}
	local pcount = #palette
	for i = 1, #text do
		local ch = text:sub(i,i)
		local color = palette[((i-1) % pcount) + 1]
		if ch == "<" then ch = "&lt;" elseif ch == ">" then ch = "&gt;" end
		table.insert(parts, ('<font color="%s">%s</font>'):format(color, ch == " " and " " or ch))
	end
	return table.concat(parts)
end

local palette = {
	"#BDBDBD",
	"#D7BFA8",
}

footer.Text = colorizePerChar("Feito por script_roube_um_brainrot", palette)

-- Estado do toggle
local isOn = false
local autoOffToken = nil

local tweenInfo = TweenInfo.new(0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local onColor = hexToColor3("#34C759")
local offColor = hexToColor3("#FF3B30")
local bgOn = hexToColor3("#0F2F0F")
local bgOff = Color3.fromRGB(34,34,36)

local function updateVisuals()
	local targetColor = isOn and onColor or offColor
	local bgTarget = isOn and bgOn or bgOff
	local textTo = isOn and "on" or "off"
	TweenService:Create(toggleButton, tweenInfo, {BackgroundColor3 = targetColor}):Play()
	TweenService:Create(mainFrame, tweenInfo, {BackgroundColor3 = bgTarget}):Play()
	toggleButton.Text = textTo
end

local function scheduleAutoOff(seconds)
	local token = {}
	autoOffToken = token
	task.delay(seconds, function()
		if autoOffToken == token and isOn then
			isOn = false
			updateVisuals()
			-- esconde notificação quando o auto-off ocorrer
			hideNotification()
		end
	end)
end

-- Clique do botão: alterna estado e, se ativado, agenda auto-off após 3s
toggleButton.MouseButton1Click:Connect(function()
	isOn = not isOn
	updateVisuals()
	if isOn then
		-- quando ligar, agenda desligar em 3 segundos
		scheduleAutoOff(3)
		showNotification()
	else
		-- quando desligar manualmente, cancela qualquer agendamento e esconde notificação
		autoOffToken = nil
		notifAutoHideToken = nil
		hideNotification()
	end
end)

-- Estado inicial
isOn = false
autoOffToken = nil
updateVisuals()

-- Implementação de arrastar (drag) do mainFrame (suporta mouse e touch)
local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

mainFrame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging and dragStart and startPos then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- Mantém a GUI visível na viewport se resolução mudar
local function clampToViewport()
	local vp = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or Vector2.new(1280,720)
	local frameSize = Vector2.new(mainFrame.AbsoluteSize.X, mainFrame.AbsoluteSize.Y)
	local pos = mainFrame.AbsolutePosition
	local newX = math.clamp(pos.X, 0, vp.X - frameSize.X)
	local newY = math.clamp(pos.Y, 0, vp.Y - frameSize.Y)
	mainFrame.Position = UDim2.new(0, newX, 0, newY)
end

UserInputService.WindowSizeChanged:Connect(function()
	task.wait(0.05)
	clampToViewport()
end)

-- garante notificação oculta inicialmente
notification.Visible = false
