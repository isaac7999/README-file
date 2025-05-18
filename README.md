local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local humanoid = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")
local destino = root.Position

-- Função para encontrar o assento de carro mais próximo
local function getNearestVehicleSeat()
	local closestSeat, minDist = nil, math.huge
	for _, v in ipairs(workspace:GetDescendants()) do
		if v:IsA("VehicleSeat") and not v.Occupant then
			local dist = (v.Position - destino).Magnitude
			if dist < minDist then
				minDist = dist
				closestSeat = v
			end
		end
	end
	return closestSeat
end

-- Função para ancorar/desancorar partes do veículo
local function setAnchored(model, state)
	for _, part in ipairs(model:GetDescendants()) do
		if part:IsA("BasePart") then
			part.Anchored = state
		end
	end
end

-- Função principal
local function puxarCarro()
	local seat = getNearestVehicleSeat()
	if not seat then
		warn("Nenhum assento encontrado.")
		return
	end

	local vehicle = seat:FindFirstAncestorOfClass("Model")
	if not vehicle or not vehicle.PrimaryPart then
		warn("Veículo inválido ou sem PrimaryPart.")
		return
	end

	-- Teleportar personagem para perto do assento
	root.CFrame = seat.CFrame * CFrame.new(0, 2.5, 0)
	task.wait(0.3)

	-- Forçar assento a sentar no humanoide
	humanoid:ChangeState(Enum.HumanoidStateType.Seated)
	seat:Sit(humanoid)

	-- Espera confirmação do assento
	local timeout = 2
	while seat.Occupant ~= humanoid and timeout > 0 do
		seat:Sit(humanoid)
		task.wait(0.2)
		timeout -= 0.2
	end

	if seat.Occupant == humanoid then
		-- Teleportar o carro todo para a posição anterior do jogador
		setAnchored(vehicle, true)
		task.wait(0.1)
		vehicle:SetPrimaryPartCFrame(CFrame.new(destino + Vector3.new(0, 2.5, 0)))
		task.wait(0.2)
		setAnchored(vehicle, false)
	else
		warn("Não foi possível assumir o controle do assento.")
	end
end

-- Executar a função
puxarCarro()
