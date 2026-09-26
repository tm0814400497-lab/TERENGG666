--============================================================
-- JADE_MAN_A AIRCRAFT CONTROL
-- FULL MOBILE VERSION
-- VERSION v2.60.5
--============================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local Stats = game:GetService("Stats")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local VERSION = "v2.60.5"

--============================================================
-- SETTINGS
--============================================================

local FlyEnabled = false
local XRayEnabled = false
local WalkEnabled = false
local AIEnabled = false
local ProximityEnabled = false
local LockEnabled = false
local OptimizerEnabled = false

local FlySpeed = 80
local WalkSpeed = 16

local AIHP = 50
local AIDistance = 18

local LockDistance = 100
local LockStrength = 70

local LockPart = "Head"

-- Target Lock:
-- "All"      = ล็อกทุกคน
-- "Selected" = ล็อกเฉพาะรายชื่อที่เลือก
local LockMode = "All"

local SelectedPlayers = {}
local CurrentTarget = nil

local OptimizerLevel = 0

local MIN_FLY = 1
local MAX_FLY = 500

local MIN_WALK = 1
local MAX_WALK = 500

local MIN_AI_HP = 1
local MAX_AI_HP = 100

local MIN_AI_DISTANCE = 5
local MAX_AI_DISTANCE = 100

local MIN_LOCK_DISTANCE = 5
local MAX_LOCK_DISTANCE = 500

local MIN_LOCK_STRENGTH = 0
local MAX_LOCK_STRENGTH = 100

local MIN_OPTIMIZER = 0
local MAX_OPTIMIZER = 100

local PROXIMITY_DISTANCE = 30

--============================================================
-- COLORS
--============================================================

local Theme = Color3.fromRGB(45,190,80)
local Accent = Color3.fromRGB(60,235,190)

local Background = Color3.fromRGB(25,25,30)
local Panel = Color3.fromRGB(39,39,47)
local Button = Color3.fromRGB(49,49,59)

local White = Color3.fromRGB(255,255,255)
local Gray = Color3.fromRGB(170,170,175)
local Red = Color3.fromRGB(210,55,55)

--============================================================
-- CHARACTER
--============================================================

local Character
local Humanoid
local RootPart
local OriginalWalkSpeed = 16

local function SetupCharacter(character)

	Character = character

	Humanoid = character:WaitForChild("Humanoid",10)

	RootPart = character:WaitForChild(
		"HumanoidRootPart",
		10
	)

	if Humanoid then
		OriginalWalkSpeed = Humanoid.WalkSpeed

		if WalkEnabled then
			Humanoid.WalkSpeed = WalkSpeed
		end
	end
end

if LocalPlayer.Character then
	task.spawn(function()
		SetupCharacter(LocalPlayer.Character)
	end)
end

LocalPlayer.CharacterAdded:Connect(function(character)

	task.wait(0.5)

	SetupCharacter(character)

	CurrentTarget = nil

	if FlyEnabled then
		task.wait(0.1)
		StartFly()
	end
end)

--============================================================
-- GUI
--============================================================

local old = PlayerGui:FindFirstChild("JADE_MAN_A_v2605")

if old then
	old:Destroy()
end

local Gui = Instance.new("ScreenGui")

Gui.Name = "JADE_MAN_A_v2605"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.DisplayOrder = 999
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = PlayerGui

--============================================================
-- HELPERS
--============================================================

local function Round(object,radius)

	local c = Instance.new("UICorner")

	c.CornerRadius = UDim.new(0,radius or 10)
	c.Parent = object

end

local function Border(object)

	local s = Instance.new("UIStroke")

	s.Color = Accent
	s.Thickness = 1
	s.Transparency = 0.15
	s.Parent = object

	return s
end

local function MakeButton(parent,text,height)

	local b = Instance.new("TextButton")

	b.Size = UDim2.new(1,-8,0,height or 42)
	b.BackgroundColor3 = Button
	b.BorderSizePixel = 0
	b.Text = text
	b.TextColor3 = White
	b.TextSize = 14
	b.Font = Enum.Font.GothamSemibold
	b.AutoButtonColor = false
	b.Parent = parent

	Round(b,10)

	return b
end

--============================================================
-- MINI HOLDER
--============================================================

local MiniHolder = Instance.new("Frame")

MiniHolder.Name = "MiniHolder"
MiniHolder.Size = UDim2.fromOffset(235,52)
MiniHolder.Position = UDim2.fromOffset(10,55)
MiniHolder.BackgroundTransparency = 1
MiniHolder.Active = true
MiniHolder.Parent = Gui

--============================================================
-- OPEN BUTTON
--============================================================

local OpenButton = Instance.new("TextButton")

OpenButton.Size = UDim2.fromOffset(48,48)
OpenButton.Position = UDim2.fromOffset(0,0)
OpenButton.BackgroundColor3 = Background
OpenButton.BorderSizePixel = 0
OpenButton.Text = "+"
OpenButton.TextColor3 = White
OpenButton.TextSize = 28
OpenButton.Font = Enum.Font.GothamBold
OpenButton.AutoButtonColor = false
OpenButton.Parent = MiniHolder

Round(OpenButton,12)

local OpenBorder = Border(OpenButton)

--============================================================
-- MINI FPS / PING
--============================================================

local MiniInfo = Instance.new("Frame")

MiniInfo.Size = UDim2.fromOffset(125,48)
MiniInfo.Position = UDim2.fromOffset(54,0)
MiniInfo.BackgroundColor3 = Background
MiniInfo.BorderSizePixel = 0
MiniInfo.Parent = MiniHolder

Round(MiniInfo,12)

local MiniFPS = Instance.new("TextLabel")

MiniFPS.Size = UDim2.new(1,0,0.5,0)
MiniFPS.BackgroundTransparency = 1
MiniFPS.Text = "FPS: --"
MiniFPS.TextColor3 = Theme
MiniFPS.TextSize = 13
MiniFPS.Font = Enum.Font.GothamBold
MiniFPS.Parent = MiniInfo

local MiniPing = Instance.new("TextLabel")

MiniPing.Size = UDim2.new(1,0,0.5,0)
MiniPing.Position = UDim2.fromScale(0,0.5)
MiniPing.BackgroundTransparency = 1
MiniPing.Text = "Ping: --"
MiniPing.TextColor3 = Theme
MiniPing.TextSize = 13
MiniPing.Font = Enum.Font.GothamBold
MiniPing.Parent = MiniInfo

--============================================================
-- MAIN
--============================================================

local Main = Instance.new("Frame")

Main.Name = "Main"
Main.Size = UDim2.fromOffset(310,285)
Main.Position = MiniHolder.Position
Main.BackgroundColor3 = Background
Main.BorderSizePixel = 0
Main.Visible = false
Main.ClipsDescendants = true
Main.Parent = Gui

Round(Main,14)

local MainBorder = Border(Main)

--============================================================
-- HEADER
--============================================================

local Header = Instance.new("Frame")

Header.Size = UDim2.new(1,0,0,60)
Header.BackgroundColor3 = Panel
Header.BorderSizePixel = 0
Header.Parent = Main

Round(Header,14)

local Avatar = Instance.new("ImageLabel")

Avatar.Size = UDim2.fromOffset(42,42)
Avatar.Position = UDim2.fromOffset(8,9)
Avatar.BackgroundColor3 = Button
Avatar.BorderSizePixel = 0
Avatar.Parent = Header

Round(Avatar,21)

task.spawn(function()

	local ok,image = pcall(function()

		return Players:GetUserThumbnailAsync(
			LocalPlayer.UserId,
			Enum.ThumbnailType.HeadShot,
			Enum.ThumbnailSize.Size100x100
		)

	end)

	if ok and image then
		Avatar.Image = image
	end

end)

local NameLabel = Instance.new("TextLabel")

NameLabel.Size = UDim2.fromOffset(145,22)
NameLabel.Position = UDim2.fromOffset(58,7)
NameLabel.BackgroundTransparency = 1
NameLabel.Text = LocalPlayer.DisplayName
NameLabel.TextColor3 = White
NameLabel.TextSize = 14
NameLabel.Font = Enum.Font.GothamBold
NameLabel.TextXAlignment = Enum.TextXAlignment.Left
NameLabel.Parent = Header

local UserLabel = Instance.new("TextLabel")

UserLabel.Size = UDim2.fromOffset(145,20)
UserLabel.Position = UDim2.fromOffset(58,29)
UserLabel.BackgroundTransparency = 1
UserLabel.Text = "@"..LocalPlayer.Name
UserLabel.TextColor3 = Gray
UserLabel.TextSize = 11
UserLabel.Font = Enum.Font.Gotham
UserLabel.TextXAlignment = Enum.TextXAlignment.Left
UserLabel.Parent = Header

local VersionLabel = Instance.new("TextLabel")

VersionLabel.Size = UDim2.fromOffset(75,24)
VersionLabel.Position = UDim2.new(1,-122,0,5)
VersionLabel.BackgroundTransparency = 1
VersionLabel.Text = VERSION
VersionLabel.TextColor3 = Accent
VersionLabel.TextSize = 16
VersionLabel.Font = Enum.Font.GothamBold
VersionLabel.Parent = Header

local MinButton = Instance.new("TextButton")

MinButton.Size = UDim2.fromOffset(42,42)
MinButton.Position = UDim2.new(1,-49,0,9)
MinButton.BackgroundColor3 = Red
MinButton.BorderSizePixel = 0
MinButton.Text = "−"
MinButton.TextColor3 = White
MinButton.TextSize = 25
MinButton.Font = Enum.Font.GothamBold
MinButton.Parent = Header

Round(MinButton,10)

--============================================================
-- MAIN SCROLL
--============================================================

local Scroll = Instance.new("ScrollingFrame")

Scroll.Size = UDim2.new(1,-12,1,-68)
Scroll.Position = UDim2.fromOffset(6,64)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 5
Scroll.ScrollBarImageColor3 = Accent
Scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
Scroll.CanvasSize = UDim2.new()
Scroll.Parent = Main

local Layout = Instance.new("UIListLayout")

Layout.Padding = UDim.new(0,7)
Layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
Layout.Parent = Scroll

--============================================================
-- PAGE
--============================================================

local Page = Instance.new("Frame")

Page.Name = "Page"
Page.Size = Main.Size
Page.Position = Main.Position
Page.BackgroundColor3 = Background
Page.BorderSizePixel = 0
Page.Visible = false
Page.ClipsDescendants = true
Page.Parent = Gui

Round(Page,14)

local PageBorder = Border(Page)

local PageHeader = Instance.new("Frame")

PageHeader.Size = UDim2.new(1,0,0,52)
PageHeader.BackgroundColor3 = Panel
PageHeader.BorderSizePixel = 0
PageHeader.Parent = Page

Round(PageHeader,14)

local Back = MakeButton(PageHeader,"← กลับ",38)

Back.Size = UDim2.fromOffset(78,38)
Back.Position = UDim2.fromOffset(7,7)

local PageTitle = Instance.new("TextLabel")

PageTitle.Size = UDim2.fromOffset(130,52)
PageTitle.Position = UDim2.fromOffset(93,0)
PageTitle.BackgroundTransparency = 1
PageTitle.Text = "Feature"
PageTitle.TextColor3 = White
PageTitle.TextSize = 15
PageTitle.Font = Enum.Font.GothamBold
PageTitle.TextXAlignment = Enum.TextXAlignment.Left
PageTitle.Parent = PageHeader

local Toggle = Instance.new("TextButton")

Toggle.Size = UDim2.fromOffset(65,35)
Toggle.Position = UDim2.new(1,-73,0,8)
Toggle.BackgroundColor3 = Red
Toggle.BorderSizePixel = 0
Toggle.Text = "OFF"
Toggle.TextColor3 = White
Toggle.TextSize = 12
Toggle.Font = Enum.Font.GothamBold
Toggle.Visible = false
Toggle.Parent = PageHeader

Round(Toggle,9)

local PageScroll = Instance.new("ScrollingFrame")

PageScroll.Size = UDim2.new(1,-12,1,-60)
PageScroll.Position = UDim2.fromOffset(6,58)
PageScroll.BackgroundTransparency = 1
PageScroll.BorderSizePixel = 0
PageScroll.ScrollBarThickness = 5
PageScroll.ScrollBarImageColor3 = Accent
PageScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
PageScroll.CanvasSize = UDim2.new()
PageScroll.Parent = Page

local PageLayout = Instance.new("UIListLayout")

PageLayout.Padding = UDim.new(0,7)
PageLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
PageLayout.Parent = PageScroll

--============================================================
-- PAGE HELPERS
--============================================================

local CurrentToggle = nil

local function ClearPage()

	for _,v in ipairs(PageScroll:GetChildren()) do

		if not v:IsA("UIListLayout") then
			v:Destroy()
		end

	end

end

local function Info(text,height)

	local label = Instance.new("TextLabel")

	label.Size = UDim2.new(1,-8,0,height or 42)
	label.BackgroundColor3 = Panel
	label.BorderSizePixel = 0
	label.Text = text
	label.TextColor3 = White
	label.TextSize = 13
	label.Font = Enum.Font.Gotham
	label.TextWrapped = true
	label.Parent = PageScroll

	Round(label,10)

	return label
end

local function PageButton(text)

	return MakeButton(PageScroll,text,40)

end

local function NumberBox(title,getter,setter,min,max)

	local frame = Instance.new("Frame")

	frame.Size = UDim2.new(1,-8,0,48)
	frame.BackgroundColor3 = Panel
	frame.BorderSizePixel = 0
	frame.Parent = PageScroll

	Round(frame,10)

	local label = Instance.new("TextLabel")

	label.Size = UDim2.new(1,-95,1,0)
	label.Position = UDim2.fromOffset(10,0)
	label.BackgroundTransparency = 1
	label.Text = title
	label.TextColor3 = White
	label.TextSize = 13
	label.Font = Enum.Font.GothamBold
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = frame

	local box = Instance.new("TextBox")

	box.Size = UDim2.fromOffset(72,34)
	box.Position = UDim2.new(1,-82,0,7)
	box.BackgroundColor3 = Background
	box.BorderSizePixel = 0
	box.Text = tostring(getter())
	box.TextColor3 = White
	box.TextSize = 13
	box.Font = Enum.Font.GothamBold
	box.ClearTextOnFocus = false
	box.Parent = frame

	Round(box,8)

	box.FocusLost:Connect(function()

		local value = tonumber(box.Text)

		if not value then
			box.Text = tostring(getter())
			return
		end

		value = math.clamp(
			math.floor(value),
			min,
			max
		)

		setter(value)
		box.Text = tostring(value)

	end)

end

local function Stepper(title,getter,setter,min,max)

	local frame = Instance.new("Frame")

	frame.Size = UDim2.new(1,-8,0,48)
	frame.BackgroundColor3 = Panel
	frame.BorderSizePixel = 0
	frame.Parent = PageScroll

	Round(frame,10)

	local label = Instance.new("TextLabel")

	label.Size = UDim2.new(1,-125,1,0)
	label.Position = UDim2.fromOffset(10,0)
	label.BackgroundTransparency = 1
	label.Text = title
	label.TextColor3 = White
	label.TextSize = 13
	label.Font = Enum.Font.GothamBold
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = frame

	local minus = Instance.new("TextButton")

	minus.Size = UDim2.fromOffset(32,32)
	minus.Position = UDim2.new(1,-108,0,8)
	minus.BackgroundColor3 = Button
	minus.BorderSizePixel = 0
	minus.Text = "−"
	minus.TextColor3 = White
	minus.TextSize = 20
	minus.Font = Enum.Font.GothamBold
	minus.Parent = frame

	Round(minus,8)

	local valueLabel = Instance.new("TextLabel")

	valueLabel.Size = UDim2.fromOffset(38,32)
	valueLabel.Position = UDim2.new(1,-70,0,8)
	valueLabel.BackgroundColor3 = Background
	valueLabel.TextColor3 = White
	valueLabel.TextSize = 13
	valueLabel.Font = Enum.Font.GothamBold
	valueLabel.Parent = frame

	Round(valueLabel,8)

	local plus = Instance.new("TextButton")

	plus.Size = UDim2.fromOffset(32,32)
	plus.Position = UDim2.new(1,-34,0,8)
	plus.BackgroundColor3 = Button
	plus.BorderSizePixel = 0
	plus.Text = "+"
	plus.TextColor3 = White
	plus.TextSize = 20
	plus.Font = Enum.Font.GothamBold
	plus.Parent = frame

	Round(plus,8)

	local function Update()
		valueLabel.Text = tostring(getter())
	end

	minus.Activated:Connect(function()

		setter(math.clamp(
			getter()-1,
			min,
			max
		))

		Update()

	end)

	plus.Activated:Connect(function()

		setter(math.clamp(
			getter()+1,
			min,
			max
		))

		Update()

	end)

	Update()

end

--============================================================
-- OPEN PAGE
--============================================================

local function OpenPage(title,getter,setter)

	ClearPage()

	PageTitle.Text = title

	if getter and setter then

		Toggle.Visible = true

		local function UpdateToggle()

			if getter() then
				Toggle.Text = "ON"
				Toggle.BackgroundColor3 = Theme
			else
				Toggle.Text = "OFF"
				Toggle.BackgroundColor3 = Red
			end

		end

		UpdateToggle()

		CurrentToggle = function()

			setter(not getter())
			UpdateToggle()

		end

	else

		Toggle.Visible = false
		CurrentToggle = nil

	end

	Page.Position = Main.Position
	Page.Visible = true
	Main.Visible = false
	MiniHolder.Visible = false

end

Toggle.Activated:Connect(function()

	if CurrentToggle then
		CurrentToggle()
	end

end)

--============================================================
-- FLY
--============================================================

local FlyVelocity
local FlyGyro

local function StopFly()

	if FlyVelocity then
		FlyVelocity:Destroy()
		FlyVelocity = nil
	end

	if FlyGyro then
		FlyGyro:Destroy()
		FlyGyro = nil
	end

	if Humanoid then
		Humanoid.AutoRotate = true
	end

end

local function StartFly()

	if not RootPart then
		return
	end

	StopFly()

	FlyVelocity = Instance.new("BodyVelocity")

	FlyVelocity.MaxForce = Vector3.new(
		math.huge,
		math.huge,
		math.huge
	)

	FlyVelocity.P = 10000
	FlyVelocity.Velocity = Vector3.zero
	FlyVelocity.Parent = RootPart

	FlyGyro = Instance.new("BodyGyro")

	FlyGyro.MaxTorque = Vector3.new(
		0,
		1000000,
		0
	)

	FlyGyro.P = 50000
	FlyGyro.D = 500
	FlyGyro.Parent = RootPart

	if Humanoid then
		Humanoid.AutoRotate = false
	end

end

local function UpdateFly()

	if not FlyEnabled then
		return
	end

	if not RootPart or not Humanoid then
		return
	end

	if not FlyVelocity or not FlyGyro then
		StartFly()
	end

	local camera = Workspace.CurrentCamera

	if not camera then
		return
	end

	local move = Humanoid.MoveDirection

	if move.Magnitude > 0 then

		FlyVelocity.Velocity =
			move.Unit * FlySpeed

	else

		FlyVelocity.Velocity =
			Vector3.zero

	end

	local look = camera.CFrame.LookVector

	local flat = Vector3.new(
		look.X,
		0,
		look.Z
	)

	if flat.Magnitude > 0.01 then

		FlyGyro.CFrame = CFrame.lookAt(
			RootPart.Position,
			RootPart.Position + flat.Unit
		)

	end

end

--============================================================
-- X-RAY
--============================================================

local XRayParts = {}

local function ClearXRay()

	for part,oldTransparency in pairs(XRayParts) do

		if part and part.Parent then

			part.LocalTransparencyModifier =
				oldTransparency

		end

	end

	table.clear(XRayParts)

end

local function ApplyXRay()

	ClearXRay()

	if not XRayEnabled then
		return
	end

	for _,obj in ipairs(Workspace:GetDescendants()) do

		if obj:IsA("BasePart") then

			if not Character or
				not obj:IsDescendantOf(Character) then

				XRayParts[obj] =
					obj.LocalTransparencyModifier

				obj.LocalTransparencyModifier =
					0.55

			end

		end

	end

end

--============================================================
-- TARGET LOCK UI
--============================================================

local TargetUI = Instance.new("Frame")

TargetUI.Name = "TargetLockUI"
TargetUI.Size = UDim2.fromOffset(130,105)
TargetUI.Position = UDim2.new(0.5,-65,0.5,-52)
TargetUI.BackgroundTransparency = 1
TargetUI.Visible = false
TargetUI.ZIndex = 100
TargetUI.Parent = Gui

local TargetCircle = Instance.new("Frame")

TargetCircle.Size = UDim2.fromOffset(54,54)
TargetCircle.Position = UDim2.fromOffset(38,0)
TargetCircle.BackgroundTransparency = 1
TargetCircle.BorderSizePixel = 0
TargetCircle.Parent = TargetUI

Round(TargetCircle,100)

local CircleStroke = Instance.new("UIStroke")

CircleStroke.Color = Accent
CircleStroke.Thickness = 2
CircleStroke.Parent = TargetCircle

local TargetName = Instance.new("TextLabel")

TargetName.Size = UDim2.fromOffset(130,22)
TargetName.Position = UDim2.fromOffset(0,58)
TargetName.BackgroundTransparency = 1
TargetName.Text = ""
TargetName.TextColor3 = White
TargetName.TextSize = 11
TargetName.Font = Enum.Font.GothamBold
TargetName.TextTruncate = Enum.TextTruncate.AtEnd
TargetName.Parent = TargetUI

local TargetUser = Instance.new("TextLabel")

TargetUser.Size = UDim2.fromOffset(130,16)
TargetUser.Position = UDim2.fromOffset(0,78)
TargetUser.BackgroundTransparency = 1
TargetUser.Text = ""
TargetUser.TextColor3 = Gray
TargetUser.TextSize = 9
TargetUser.Font = Enum.Font.Gotham
TargetUser.TextTruncate = Enum.TextTruncate.AtEnd
TargetUser.Parent = TargetUI

--============================================================
-- TARGET LOCK
--============================================================

local function GetTargetPart(player)

	if not player or not player.Character then
		return nil
	end

	if LockPart == "Head" then

		return player.Character:FindFirstChild("Head")

	end

	return player.Character:FindFirstChild(
		"HumanoidRootPart"
	)

end

local function IsSelected(player)

	return SelectedPlayers[player] == true

end

local function ValidTarget(player)

	if not player or player == LocalPlayer then
		return false
	end

	if LockMode == "Selected" and
		not IsSelected(player) then

		return false
	end

	local char = player.Character

	if not char then
		return false
	end

	local hum = char:FindFirstChildOfClass("Humanoid")
	local part = GetTargetPart(player)

	if not hum or not part then
		return false
	end

	if hum.Health <= 0 then
		return false
	end

	return true
end

-- ตรวจจับรอบตัว 360 องศา
-- ไม่ใช้มุมกล้องเป็นเงื่อนไข
local function FindTarget()

	if not RootPart then
		return nil
	end

	local nearest = nil
	local nearestDistance = LockDistance

	for _,player in ipairs(Players:GetPlayers()) do

		if ValidTarget(player) then

			local part = GetTargetPart(player)

			if part then

				local distance =
					(RootPart.Position - part.Position).Magnitude

				if distance <= nearestDistance then

					nearestDistance = distance
					nearest = player

				end

			end

		end

	end

	return nearest
end

local function UpdateTargetUI()

	if not LockEnabled then

		TargetUI.Visible = false
		TargetName.Text = ""
		TargetUser.Text = ""

		return
	end

	TargetUI.Visible = true

	if CurrentTarget and ValidTarget(CurrentTarget) then

		TargetName.Text =
			CurrentTarget.DisplayName

		TargetUser.Text =
			"@"..CurrentTarget.Name

	else

		TargetName.Text = "Target: None"
		TargetUser.Text = ""

	end

end

local function UpdateTargetLock()

	if not LockEnabled then

		CurrentTarget = nil

		UpdateTargetUI()

		return
	end

	if not RootPart then
		return
	end

	-- เป้าหมายเดิมใช้ได้อยู่หรือไม่
	if CurrentTarget then

		if not ValidTarget(CurrentTarget) then
			CurrentTarget = nil
		else

			local currentPart =
				GetTargetPart(CurrentTarget)

			if currentPart then

				local currentDistance =
					(
						RootPart.Position -
						currentPart.Position
					).Magnitude

				if currentDistance >
					LockDistance then

					CurrentTarget = nil

				end

			end

		end

	end

	-- ถ้าไม่มีเป้าหมาย ให้หาใหม่
	if not CurrentTarget then

		CurrentTarget = FindTarget()

	end

	if not CurrentTarget then

		UpdateTargetUI()
		return
	end

	local targetPart =
		GetTargetPart(CurrentTarget)

	local targetHum =
		CurrentTarget.Character and
		CurrentTarget.Character:
			FindFirstChildOfClass("Humanoid")

	if not targetPart or
		not targetHum or
		targetHum.Health <= 0 then

		CurrentTarget = nil
		UpdateTargetUI()
		return
	end

	local distance =
		(
			RootPart.Position -
			targetPart.Position
		).Magnitude

	if distance > LockDistance then

		CurrentTarget = FindTarget()
		UpdateTargetUI()
		return

	end

	local alpha =
		math.clamp(
			LockStrength / 100,
			0,
			1
		)

	local camera =
		Workspace.CurrentCamera

	-- หมุนกล้องหาเป้าหมาย
	if camera then

		local desired =
			CFrame.lookAt(
				camera.CFrame.Position,
				targetPart.Position
			)

		camera.CFrame =
			camera.CFrame:Lerp(
				desired,
				alpha
			)

	end

	-- หมุนตัวละครในแนวราบ
	if Humanoid and
		RootPart then

		Humanoid.AutoRotate = false

		local flatTarget =
			Vector3.new(
				targetPart.Position.X,
				RootPart.Position.Y,
				targetPart.Position.Z
			)

		local flatDirection =
			flatTarget -
			RootPart.Position

		if flatDirection.Magnitude > 0.01 then

			RootPart.CFrame =
				RootPart.CFrame:Lerp(
					CFrame.lookAt(
						RootPart.Position,
						flatTarget
					),
					alpha
				)

		end

	end

	UpdateTargetUI()

end

--============================================================
-- PROXIMITY
--============================================================

local Highlights = {}

local function ClearHighlights()

	for player,object in pairs(Highlights) do

		if object then
			object:Destroy()
		end

		Highlights[player] = nil

	end

end

local function UpdateProximity()

	if not ProximityEnabled then

		ClearHighlights()
		return

	end

	if not RootPart then
		return
	end

	for _,player in ipairs(Players:GetPlayers()) do

		if player ~= LocalPlayer then

			local char = player.Character

			if char then

				local hum =
					char:FindFirstChildOfClass("Humanoid")

				local hrp =
					char:FindFirstChild("HumanoidRootPart")

				if hum and hrp and hum.Health > 0 then

					local distance =
						(
							RootPart.Position -
							hrp.Position
						).Magnitude

					if distance <= PROXIMITY_DISTANCE then

						if not Highlights[player] then

							local h =
								Instance.new("Highlight")

							h.Adornee = char
							h.DepthMode =
								Enum.HighlightDepthMode.AlwaysOnTop

							h.FillTransparency = 0.65
							h.OutlineTransparency = 0
							h.Parent = Gui

							Highlights[player] = h

						end

						if hrp.AssemblyLinearVelocity.Magnitude > 1 then

							Highlights[player].FillColor =
								Color3.fromRGB(0,0,0)

						else

							Highlights[player].FillColor =
								Color3.fromRGB(80,80,80)

						end

						Highlights[player].OutlineColor =
							Accent

					elseif Highlights[player] then

						Highlights[player]:Destroy()
						Highlights[player] = nil

					end

				end

			end

		end

	end

end

--============================================================
-- AI ESCAPE
--============================================================

local function FindNearbyPlayer()

	if not RootPart then
		return nil
	end

	local nearest
	local nearestDistance = AIDistance

	for _,player in ipairs(Players:GetPlayers()) do

		if player ~= LocalPlayer then

			local char = player.Character

			if char then

				local hum =
					char:FindFirstChildOfClass("Humanoid")

				local hrp =
					char:FindFirstChild("HumanoidRootPart")

				if hum and hrp and hum.Health > 0 then

					local distance =
						(
							RootPart.Position -
							hrp.Position
						).Magnitude

					if distance <= nearestDistance then

						nearestDistance = distance
						nearest = player

					end

				end

			end

		end

	end

	return nearest
end

local function UpdateAI()

	if not AIEnabled then
		return
	end

	if not Humanoid or not RootPart then
		return
	end

	if Humanoid.Health <= 0 then
		return
	end

	local nearby = FindNearbyPlayer()

	local lowHP =
		Humanoid.Health <= AIHP

	if not nearby and not lowHP then
		return
	end

	local direction = Vector3.zero

	if nearby and nearby.Character then

		local enemyRoot =
			nearby.Character:FindFirstChild(
				"HumanoidRootPart"
			)

		if enemyRoot then

			local away =
				RootPart.Position -
				enemyRoot.Position

			direction =
				Vector3.new(
					away.X,
					0,
					away.Z
				)

			if direction.Magnitude > 0 then
				direction = direction.Unit
			end

		end

	end

	if direction.Magnitude <= 0 then

		direction =
			-RootPart.CFrame.LookVector

		direction =
			Vector3.new(
				direction.X,
				0,
				direction.Z
			)

		if direction.Magnitude > 0 then
			direction = direction.Unit
		end

	end

	local params = RaycastParams.new()

	params.FilterType =
		Enum.RaycastFilterType.Exclude

	params.FilterDescendantsInstances = {
		Character
	}

	local result =
		Workspace:Raycast(
			RootPart.Position +
				Vector3.new(0,1,0),
			direction * 5,
			params
		)

	if result then

		direction =
			Vector3.new(
				-direction.Z,
				0,
				direction.X
			)

	end

	Humanoid:Move(direction,false)

end

--============================================================
-- FPS
--============================================================

local FPS = 60
local FrameCount = 0
local LastFPS = os.clock()

RunService.RenderStepped:Connect(function()

	FrameCount += 1

	local now = os.clock()

	if now - LastFPS >= 1 then

		FPS = FrameCount
		FrameCount = 0
		LastFPS = now

		MiniFPS.Text =
			"FPS: "..FPS

	end

end)

--============================================================
-- PING
--============================================================

task.spawn(function()

	while Gui.Parent do

		local ping = "--"

		pcall(function()

			local network = Stats.Network

			local item =
				network.ServerStatsItem["Data Ping"]

			local value = item:GetValue()

			if value then

				ping =
					tostring(
						math.floor(value)
					).." ms"

			end

		end)

		MiniPing.Text =
			"Ping: "..ping

		task.wait(1)

	end

end)

--============================================================
-- FPS OPTIMIZER
--============================================================

local OriginalShadows =
	Lighting.GlobalShadows

local OriginalBrightness =
	Lighting.Brightness

local OriginalExposure =
	Lighting.ExposureCompensation

local function ApplyOptimizer()

	if not OptimizerEnabled then

		Lighting.GlobalShadows =
			OriginalShadows

		Lighting.Brightness =
			OriginalBrightness

		Lighting.ExposureCompensation =
			OriginalExposure

		return
	end

	if OptimizerLevel >= 20 then
		Lighting.GlobalShadows = false
	end

	if OptimizerLevel >= 40 then

		Lighting.Brightness =
			math.clamp(
				OriginalBrightness +
				OptimizerLevel/100,
				0,
				10
			)

		Lighting.ExposureCompensation =
			math.clamp(
				OriginalExposure +
				OptimizerLevel/100,
				-2,
				3
			)

	end

end

--============================================================
-- MENU
--============================================================

local function Menu(text)
	return MakeButton(Scroll,text,44)
end

local FlyButton =
	Menu("✈️  Fly")

local XRayButton =
	Menu("👁️  X-Ray")

local WalkButton =
	Menu("🏃  Walk Speed")

local AIButton =
	Menu("🤖  AI")

local ProximityButton =
	Menu("📡  Proximity")

local LockButton =
	Menu("🎯  Target Lock")

local FPSButton =
	Menu("⚡  FPS Optimizer")

local PerformanceButton =
	Menu("📊  Performance")

local SettingsButton =
	Menu("⚙️  Settings")

--============================================================
-- FLY PAGE
--============================================================

FlyButton.Activated:Connect(function()

	OpenPage(
		"✈️ Fly",
		function()
			return FlyEnabled
		end,
		function(value)

			FlyEnabled = value

			if value then
				StartFly()
			else
				StopFly()
			end

		end
	)

	Stepper(
		"Flight Speed",
		function()
			return FlySpeed
		end,
		function(value)
			FlySpeed = value
		end,
		MIN_FLY,
		MAX_FLY
	)

	Info(
		"ใช้จอยมือถือควบคุมทิศทางการบิน",
		42
	)

end)

--============================================================
-- X-RAY PAGE
--============================================================

XRayButton.Activated:Connect(function()

	OpenPage(
		"👁️ X-Ray",
		function()
			return XRayEnabled
		end,
		function(value)

			XRayEnabled = value
			ApplyXRay()

		end
	)

	Info(
		"ทำให้บล็อกโปร่งแสงเพื่อช่วยมองเห็นสิ่งที่อยู่ด้านหลัง",
		55
	)

end)

--============================================================
-- WALK PAGE
--============================================================

WalkButton.Activated:Connect(function()

	OpenPage(
		"🏃 Walk Speed",
		function()
			return WalkEnabled
		end,
		function(value)

			WalkEnabled = value

			if Humanoid then

				if value then
					Humanoid.WalkSpeed = WalkSpeed
				else
					Humanoid.WalkSpeed = OriginalWalkSpeed
				end

			end

		end
	)

	Stepper(
		"Walk Speed",
		function()
			return WalkSpeed
		end,
		function(value)

			WalkSpeed = value

			if WalkEnabled and Humanoid then
				Humanoid.WalkSpeed = WalkSpeed
			end

		end,
		MIN_WALK,
		MAX_WALK
	)

end)

--============================================================
-- AI PAGE
--============================================================

AIButton.Activated:Connect(function()

	OpenPage(
		"🤖 AI",
		function()
			return AIEnabled
		end,
		function(value)
			AIEnabled = value
		end
	)

	NumberBox(
		"❤️ HP Trigger",
		function()
			return AIHP
		end,
		function(value)
			AIHP = value
		end,
		MIN_AI_HP,
		MAX_AI_HP
	)

	Stepper(
		"ระยะหลบ",
		function()
			return AIDistance
		end,
		function(value)
			AIDistance = value
		end,
		MIN_AI_DISTANCE,
		MAX_AI_DISTANCE
	)

	Info(
		"เมื่อเลือดต่ำกว่าค่าที่กำหนด หรือมีผู้เล่นเข้ามาใกล้ AI จะพยายามเดินหลบ",
		58
	)

	Info(
		"ถ้ามีบล็อกอยู่ด้านหน้า AI จะพยายามเปลี่ยนทิศทาง",
		50
	)

end)

--============================================================
-- PROXIMITY PAGE
--============================================================

ProximityButton.Activated:Connect(function()

	OpenPage(
		"📡 Proximity",
		function()
			return ProximityEnabled
		end,
		function(value)

			ProximityEnabled = value

			if not value then
				ClearHighlights()
			end

		end
	)

	Info(
		"ตรวจจับผู้เล่นภายใน "..PROXIMITY_DISTANCE.." studs",
		42
	)

end)

--============================================================
-- TARGET LOCK PAGE
--============================================================

local function BuildTargetLockPage()

	OpenPage(
		"🎯 Target Lock",
		function()
			return LockEnabled
		end,
		function(value)

			LockEnabled = value

			if not value then

				CurrentTarget = nil
				UpdateTargetUI()

			end

		end
	)

	--========================================================
	-- LOCK PART
	--========================================================

	Info(
		"ตำแหน่งเป้าหมาย",
		35
	)

	local headButton =
		PageButton("🔴 Head")

	local bodyButton =
		PageButton("🟢 Body")

	local partInfo =
		Info(
			"กำลังล็อก: "..LockPart,
			35
		)

	local function UpdatePartInfo()
		partInfo.Text =
			"กำลังล็อก: "..LockPart
	end

	headButton.Activated:Connect(function()

		LockPart = "Head"
		CurrentTarget = nil
		UpdatePartInfo()

	end)

	bodyButton.Activated:Connect(function()

		LockPart = "Body"
		CurrentTarget = nil
		UpdatePartInfo()

	end)

	--========================================================
	-- MODE
	--========================================================

	Info(
		"โหมดล็อกเป้า",
		35
	)

	local allButton =
		PageButton(
			"👥 โหมด 1 — ล็อกทุกคน"
		)

	local selectedButton =
		PageButton(
			"👤 โหมด 2 — เลือกรายชื่อ"
		)

	local modeInfo =
		Info(
			"โหมดปัจจุบัน: "..LockMode,
			35
		)

	allButton.Activated:Connect(function()

		LockMode = "All"
		CurrentTarget = nil

		modeInfo.Text =
			"โหมดปัจจุบัน: All Players"

	end)

	selectedButton.Activated:Connect(function()

		LockMode = "Selected"
		CurrentTarget = nil

		modeInfo.Text =
			"โหมดปัจจุบัน: Selected Players"

	end)

	--========================================================
	-- PLAYER LIST
	--========================================================

	Info(
		"รายชื่อผู้เล่น — กด ON เพื่อเพิ่มเข้าเป้าหมาย",
		45
	)

	local playerCount = 0

	for _,player in ipairs(Players:GetPlayers()) do

		if player ~= LocalPlayer then

			playerCount += 1

			local row =
				Instance.new("Frame")

			row.Size =
				UDim2.new(1,-8,0,52)

			row.BackgroundColor3 =
				Panel

			row.BorderSizePixel = 0
			row.Parent = PageScroll

			Round(row,10)

			local image =
				Instance.new("ImageLabel")

			image.Size =
				UDim2.fromOffset(38,38)

			image.Position =
				UDim2.fromOffset(6,7)

			image.BackgroundColor3 =
				Button

			image.BorderSizePixel = 0
			image.Parent = row

			Round(image,19)

			task.spawn(function()

				local ok,url =
					pcall(function()

						return Players:GetUserThumbnailAsync(
							player.UserId,
							Enum.ThumbnailType.HeadShot,
							Enum.ThumbnailSize.Size100x100
						)

					end)

				if ok and url then
					image.Image = url
				end

			end)

			local display =
				Instance.new("TextLabel")

			display.Size =
				UDim2.new(1,-105,0,22)

			display.Position =
				UDim2.fromOffset(52,4)

			display.BackgroundTransparency = 1
			display.Text =
				player.DisplayName

			display.TextColor3 =
				White

			display.TextSize = 12
			display.Font =
				Enum.Font.GothamBold

			display.TextXAlignment =
				Enum.TextXAlignment.Left

			display.Parent = row

			local username =
				Instance.new("TextLabel")

			username.Size =
				UDim2.new(1,-105,0,18)

			username.Position =
				UDim2.fromOffset(52,26)

			username.BackgroundTransparency = 1
			username.Text =
				"@"..player.Name

			username.TextColor3 =
				Gray

			username.TextSize = 9
			username.Font =
				Enum.Font.Gotham

			username.TextXAlignment =
				Enum.TextXAlignment.Left

			username.Parent = row

			local selectButton =
				Instance.new("TextButton")

			selectButton.Size =
				UDim2.fromOffset(42,32)

			selectButton.Position =
				UDim2.new(1,-48,0,10)

			selectButton.BorderSizePixel = 0
			selectButton.TextColor3 = White
			selectButton.TextSize = 10
			selectButton.Font =
				Enum.Font.GothamBold

			selectButton.Parent = row

			Round(selectButton,8)

			local function UpdateSelected()

				if SelectedPlayers[player] then

					selectButton.Text = "ON"

					selectButton.BackgroundColor3 =
						Theme

				else

					selectButton.Text = "OFF"

					selectButton.BackgroundColor3 =
						Red

				end

			end

			selectButton.Activated:Connect(function()

				SelectedPlayers[player] =
					not SelectedPlayers[player]

				-- ถ้าเอาคนที่กำลังล็อกออก
				if CurrentTarget == player and
					not SelectedPlayers[player] then

					CurrentTarget = nil

				end

				UpdateSelected()

			end)

			UpdateSelected()

		end

	end

	if playerCount == 0 then

		Info(
			"ยังไม่มีผู้เล่นคนอื่นในเซิร์ฟเวอร์",
			42
		)

	end

	--========================================================
	-- SETTINGS
	--========================================================

	Stepper(
		"Lock Distance",
		function()
			return LockDistance
		end,
		function(value)

			LockDistance = value

			if CurrentTarget then

				local part =
					GetTargetPart(CurrentTarget)

				if part and RootPart then

					if (
						RootPart.Position -
						part.Position
					).Magnitude > LockDistance then

						CurrentTarget = nil

					end

				end

			end

		end,
		MIN_LOCK_DISTANCE,
		MAX_LOCK_DISTANCE
	)

	Stepper(
		"Lock Strength",
		function()
			return LockStrength
		end,
		function(value)
			LockStrength = value
		end,
		MIN_LOCK_STRENGTH,
		MAX_LOCK_STRENGTH
	)

	Info(
		"ระบบตรวจจับเป้าหมายรอบตัว 360° ทั้งด้านหน้า ด้านหลัง และด้านข้าง โดยไม่จำเป็นต้องหันกล้องไปหาเป้าหมายก่อน",
		65
	)

end

LockButton.Activated:Connect(function()
	BuildTargetLockPage()
end)

--============================================================
-- FPS OPTIMIZER PAGE
--============================================================

FPSButton.Activated:Connect(function()

	OpenPage(
		"⚡ FPS Optimizer",
		function()
			return OptimizerEnabled
		end,
		function(value)

			OptimizerEnabled = value
			ApplyOptimizer()

		end
	)

	Stepper(
		"Optimizer",
		function()
			return OptimizerLevel
		end,
		function(value)

			OptimizerLevel = value
			ApplyOptimizer()

		end,
		MIN_OPTIMIZER,
		MAX_OPTIMIZER
	)

	Info(
		"ปรับระดับการลดภาระกราฟิก",
		45
	)

end)

--============================================================
-- PERFORMANCE
--============================================================

PerformanceButton.Activated:Connect(function()

	OpenPage(
		"📊 Performance",
		nil,
		nil
	)

	local fpsLabel =
		Info(
			"FPS: "..FPS,
			40
		)

	Info(
		"Version: "..VERSION,
		40
	)

	task.spawn(function()

		while Page.Visible and
			PageTitle.Text == "📊 Performance" do

			fpsLabel.Text =
				"FPS: "..FPS

			task.wait(1)

		end

	end)

end)

--============================================================
-- SETTINGS
--============================================================

local function ApplyTheme()

	for _,object in ipairs(Gui:GetDescendants()) do

		if object:IsA("UIStroke") then
			object.Color = Accent
		end

	end

	VersionLabel.TextColor3 = Accent
	MiniFPS.TextColor3 = Theme
	MiniPing.TextColor3 = Theme

	OpenBorder.Color = Accent
	MainBorder.Color = Accent
	PageBorder.Color = Accent
	CircleStroke.Color = Accent

end

local function SetTheme(name)

	if name == "เขียว" then

		Theme = Color3.fromRGB(45,220,80)
		Accent = Color3.fromRGB(60,235,190)

	elseif name == "แดง" then

		Theme = Color3.fromRGB(255,70,70)
		Accent = Color3.fromRGB(255,130,130)

	elseif name == "น้ำเงิน" then

		Theme = Color3.fromRGB(60,130,255)
		Accent = Color3.fromRGB(100,190,255)

	elseif name == "ม่วง" then

		Theme = Color3.fromRGB(180,80,255)
		Accent = Color3.fromRGB(220,140,255)

	elseif name == "เหลือง" then

		Theme = Color3.fromRGB(255,200,50)
		Accent = Color3.fromRGB(255,230,100)

	elseif name == "ฟ้าน้ำทะเล" then

		Theme = Color3.fromRGB(40,210,190)
		Accent = Color3.fromRGB(80,255,230)

	elseif name == "ชมพู" then

		Theme = Color3.fromRGB(255,80,170)
		Accent = Color3.fromRGB(255,140,210)

	end

	ApplyTheme()

end

SettingsButton.Activated:Connect(function()

	OpenPage(
		"⚙️ Settings",
		nil,
		nil
	)

	Info(
		"JADE_MAN_A AIRCRAFT CONTROL "..VERSION,
		42
	)

	Info(
		"เลือกสี UI",
		38
	)

	local colors = {
		"เขียว",
		"แดง",
		"น้ำเงิน",
		"ม่วง",
		"เหลือง",
		"ฟ้าน้ำทะเล",
		"ชมพู"
	}

	for _,name in ipairs(colors) do

		local b =
			MakeButton(
				PageScroll,
				name,
				38
			)

		b.Activated:Connect(function()
			SetTheme(name)
		end)

	end

	local rainbow =
		MakeButton(
			PageScroll,
			"🌈 Rainbow",
			38
		)

	rainbow.Activated:Connect(function()

		task.spawn(function()

			for hue = 0,1,0.02 do

				if not Page.Visible then
					break
				end

				Theme =
					Color3.fromHSV(
						hue,
						0.8,
						1
					)

				Accent =
					Color3.fromHSV(
						(hue+0.1)%1,
						0.7,
						1
					)

				ApplyTheme()

				task.wait(0.03)

			end

		end)

	end)

end)

--============================================================
-- BACK
--============================================================

Back.Activated:Connect(function()

	Page.Visible = false
	Main.Position = MiniHolder.Position
	Main.Visible = true
	MiniHolder.Visible = false

end)

--============================================================
-- MINIMIZE
--============================================================

MinButton.Activated:Connect(function()

	Page.Visible = false
	Main.Visible = false
	MiniHolder.Visible = true

end)

--============================================================
-- OPEN BUTTON
--============================================================

local WasDragged = false
local Dragging = false
local DragStart
local StartPosition

OpenButton.Activated:Connect(function()

	if WasDragged then

		WasDragged = false
		return

	end

	Main.Position = MiniHolder.Position
	Main.Visible = true
	MiniHolder.Visible = false

end)

OpenButton.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.Touch
		or
		input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		Dragging = true
		WasDragged = false

		DragStart = input.Position
		StartPosition = MiniHolder.Position

	end

end)

UserInputService.InputChanged:Connect(function(input)

	if not Dragging then
		return
	end

	if input.UserInputType ~=
		Enum.UserInputType.Touch
		and input.UserInputType ~=
		Enum.UserInputType.MouseMovement then

		return

	end

	local delta =
		input.Position - DragStart

	if delta.Magnitude > 5 then
		WasDragged = true
	end

	local camera =
		Workspace.CurrentCamera

	if not camera then
		return
	end

	local viewport =
		camera.ViewportSize

	local x =
		StartPosition.X.Offset + delta.X

	local y =
		StartPosition.Y.Offset + delta.Y

	local maxX =
		math.max(
			0,
			viewport.X -
			MiniHolder.AbsoluteSize.X
		)

	local maxY =
		math.max(
			0,
			viewport.Y -
			MiniHolder.AbsoluteSize.Y
		)

	x = math.clamp(x,0,maxX)
	y = math.clamp(y,0,maxY)

	MiniHolder.Position =
		UDim2.fromOffset(x,y)

end)

UserInputService.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.Touch
		or
		input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		Dragging = false

	end

end)

--============================================================
-- UPDATE
--============================================================

RunService.RenderStepped:Connect(function()

	pcall(UpdateFly)
	pcall(UpdateTargetLock)
	pcall(UpdateAI)

	if WalkEnabled and Humanoid then

		pcall(function()

			Humanoid.WalkSpeed =
				WalkSpeed

		end)

	end

end)

--============================================================
-- PROXIMITY LOOP
--============================================================

task.spawn(function()

	while Gui.Parent do

		pcall(UpdateProximity)

		task.wait(0.15)

	end

end)

--============================================================
-- XRAY LOOP
--============================================================

task.spawn(function()

	while Gui.Parent do

		if XRayEnabled then
			pcall(ApplyXRay)
		end

		task.wait(1)

	end

end)

--============================================================
-- PLAYER CLEANUP
--============================================================

Players.PlayerRemoving:Connect(function(player)

	SelectedPlayers[player] = nil

	if CurrentTarget == player then

		CurrentTarget = nil
		UpdateTargetUI()

	end

	if Highlights[player] then

		Highlights[player]:Destroy()
		Highlights[player] = nil

	end

end)

--============================================================
-- PLAYER JOIN
--============================================================

Players.PlayerAdded:Connect(function(player)

	-- รายชื่อจะถูกสร้างใหม่เมื่อเปิดหน้า Target Lock
	-- จึงรองรับผู้เล่นที่เข้ามาหลังจากเปิดเกมแล้ว
end)

--============================================================
-- INITIALIZE
--============================================================

ApplyTheme()

Main.Position =
	MiniHolder.Position

Main.Visible = true
MiniHolder.Visible = false
Page.Visible = false

print(
	"[JADE_MAN_A] "..VERSION.." loaded successfully"
)
