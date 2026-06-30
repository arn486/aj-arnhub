-- [[ DEOBFUSCATED BY @mommy owner]] --

-- // [1] SERVICES //
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer

-- // [2] CONFIGURATION //
local CONFIG = {
    Names = {
        ScreenGui = "Space Anti Bat",
        MainFrame = "MainFrame",
        TitleLabel = "TitleLabel",
        ContentFrame = "ContentFrame",
        Prefix = "Feature_"
    },

    Size = {
        MainFrame = UDim2.new(0, 240, 0, 210), -- Increased height to give plenty of space for bottom text
        TitleFrame = UDim2.new(1, 0, 0, 40),
        ContentFrame = UDim2.new(1, -20, 1, -55),
        Button = UDim2.new(1, 0, 0, 40),
        RowFrame = UDim2.new(1, 0, 0, 30),
        BindButton = UDim2.new(0, 70, 1, 0)
    },

    Position = {
        MainFrame = UDim2.new(0.5, -120, 0.5, -135),
        TitleLabel = UDim2.new(0, 15, 0, 0),
        ContentFrame = UDim2.new(0, 10, 0, 45),
        BindButton = UDim2.new(0.6, 0, 0, 0)
    },

    Colors = {
        Background = Color3.fromRGB(0, 0, 0),
        ButtonBg = Color3.fromRGB(20, 20, 20),
        BindB
