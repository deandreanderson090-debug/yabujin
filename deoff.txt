
local YabujinDeobfuscated = {}

-- ============================================================================
-- PART 1: Yabujin Client (Roblox Rivals) Core Functions & Hooks
-- ============================================================================

-- ----------------------------------------------------------------------------
-- Function [22]: runWithElevatedThreadIdentity (485 bytes)
-- Description: Elevates executor thread identity to 8 (with fallback to 7, 6, 3 on 'lacking capability') for privileged Roblox API calls.
-- Resolved Upvalues: [1] = setthreadidentity, [2] = pcall, [3] = getthreadidentity
-- ----------------------------------------------------------------------------
do
    local setthreadidentity, pcall, getthreadidentity
    YabujinDeobfuscated.runWithElevatedThreadIdentity = function(b, ...)
        if not setthreadidentity then return pcall(b, ...)
    end
    local J
    pcall(function()
        J = getthreadidentity and(getthreadidentity())
    end)
    pcall(setthreadidentity, 8)
    local N = {pcall(b, ...)}
    if not N[1] and type(N[2]) == "string" and(N[2]:find("lacking capability", 1, true)) then
        local A = {7, 6, 3}
        for q, q in ipairs(A) do
            pcall(setthreadidentity, q)
            N = {pcall(b, ...)}
            if N[1] then
                break
            end
            if type(N[2]) ~= "string" or not N[2]:find("lacking capability", 1, true) then
                break
            end
        end
    end
    if J ~= nil then
        pcall(setthreadidentity, J)
    end
    return table.unpack(N)
end
end

-- ----------------------------------------------------------------------------
-- Function [125]: runSafeQueueWorker (205 bytes)
-- Description: Background coroutine loop that maintains thread identity 8 and executes queued callbacks in _safeQueue via _withId.
-- Resolved Upvalues: [1] = setthreadidentity, [2] = pcall, [3] = spoofConfig
-- ----------------------------------------------------------------------------
do
    local setthreadidentity, pcall, spoofConfig
    YabujinDeobfuscated.runSafeQueueWorker = function()
        while true do
            if setthreadidentity then
                pcall(setthreadidentity, 8)
            end
            while#spoofConfig._safeQueue > 0 do
                local b = table.remove(spoofConfig._safeQueue, 1)
                if b then
                    spoofConfig._withId(b)
                end
            end
            task.wait()
        end
    end
end

-- ----------------------------------------------------------------------------
-- Function [55]: getRealUserId (210 bytes)
-- Description: Returns the real UserId of a Player (checking LocalPlayer and RealUserIdMap before player.UserId).
-- Resolved Upvalues: [1] = LocalPlayer, [2] = localOriginalUserId, [3] = spoofConfig
-- ----------------------------------------------------------------------------
do
    local LocalPlayer, localOriginalUserId, spoofConfig
    YabujinDeobfuscated.getRealUserId = function(b)
        if b == LocalPlayer then return localOriginalUserId
    end
    local J = spoofConfig.RealUserIdMap
    if J and J[b] then
        return J[b]
    end
    if typeof(b) == "Instance" and(b:IsA("Player")) then
        return b.UserId
    end
    return 0
end
end

-- ----------------------------------------------------------------------------
-- Function [42]: getOriginalPlayerName (178 bytes)
-- Description: Returns the unspoofed username of a Player from OtherPlayerOriginals.
-- Resolved Upvalues: [1] = LocalPlayer, [2] = localOriginalName, [3] = spoofConfig
-- ----------------------------------------------------------------------------
do
    local LocalPlayer, localOriginalName, spoofConfig
    YabujinDeobfuscated.getOriginalPlayerName = function(b)
        if b == LocalPlayer then return localOriginalName
    end
    local J = spoofConfig.OtherPlayerOriginals
    if J and J[b] and J[b].Name then
        return J[b].Name
    end
    return b.Name
end
end

-- ----------------------------------------------------------------------------
-- Function [108]: getSpoofedPlayerNames (693 bytes)
-- Description: Returns (spoofedName, spoofedDisplayName) for LocalPlayer, yabujinUsers[userId].spoofer, or AnonymousMode.
-- Resolved Upvalues: [1] = LocalPlayer, [2] = spoofConfig, [3] = getRealUserId, [4] = getYabujinSpoofField
-- ----------------------------------------------------------------------------
do
    local LocalPlayer, spoofConfig, getRealUserId, getYabujinSpoofField
    YabujinDeobfuscated.getSpoofedPlayerNames = function(b)
        if not b then return nil, nil
    end
    if b == LocalPlayer then
        if not spoofConfig.UsernameSpoof then
            return nil, nil
        end
        local J, N = spoofConfig.SpoofedName, spoofConfig.SpoofedDisplayName
        local A = J and J ~= "" and J or nil
        return A, N and N ~= "" and N or A
    end
    local J = getRealUserId(b)
    b = yabujinUsers[J] or yabujinUsers[tostring(J)]
    if b and b.spoofer and(next(b.spoofer)) then
        local b = getYabujinSpoofField(J, "Username")
        if not b or type(b) ~= "table" then
            return nil, nil
        end
        local J = b.name and b.name ~= "" and b.name or nil
        return J, b.displayName and b.displayName ~= "" and b.displayName or J
    end
    if spoofConfig.AnonymousMode then
        return spoofConfig.AnonName or "Roblox", spoofConfig.AnonDisplayName or "Roblox"
    end
    return nil, nil
end
end

-- ----------------------------------------------------------------------------
-- Function [91]: applyLocalPlayerIdentitySpoof (1190 bytes)
-- Description: Applies spoofed Name, DisplayName, and UserId to LocalPlayer and updates the in-game PlayerLabel UI.
-- Resolved Upvalues: [1] = spoofConfig, [2] = origPlayerName, [3] = LocalPlayer, [4] = pcall, [5] = origDisplayName, [6] = origUserId
-- ----------------------------------------------------------------------------
do
    local spoofConfig, origPlayerName, LocalPlayer, pcall, origDisplayName, origUserId
    YabujinDeobfuscated.applyLocalPlayerIdentitySpoof = function()
        local b = nil
        if spoofConfig.UsernameSpoof then
            local J = spoofConfig.SpoofedName
            b = J and J ~= "" and J or origPlayerName
        else
            b = origPlayerName
        end
        local J = b ~= LocalPlayer.Name
        if J then
            pcall(function()
                local N = spoofConfig._findPlayersFrame and(spoofConfig._findPlayersFrame())
                if N then
                    local A = N:FindFirstChild("PlayerLabel" .. LocalPlayer.Name)
                    if A then
                        A:Destroy()
                    end
                end
            end)
        end
        pcall(function()
            LocalPlayer.Name = b
        end)
        if spoofConfig.UsernameSpoof then
            local b, N = spoofConfig.SpoofedDisplayName, spoofConfig.SpoofedName
            if b and b ~= "" then
                pcall(function()
                    LocalPlayer.DisplayName = b
                end)
            elseif N and N ~= "" then
                pcall(function()
                    LocalPlayer.DisplayName = N
                end)
            else
                pcall(function()
                    LocalPlayer.DisplayName = origDisplayName
                end)
            end
        else
            pcall(function()
                LocalPlayer.DisplayName = origDisplayName
            end)
        end
        if not J then
            pcall(function()
                local b = spoofConfig._findPlayersFrame and(spoofConfig._findPlayersFrame())
                if b then
                    local J = b:FindFirstChild("PlayerLabel" .. LocalPlayer.Name)
                    if J then
                        local b = J:FindFirstChild("DisplayNameLabel")
                        if b then
                            b.Text = LocalPlayer.DisplayName
                        end
                    end
                end
            end)
        end
        spoofConfig._withId(function()
            if spoofConfig.AvatarSpoof and spoofConfig.AvatarUserId and spoofConfig.AvatarUserId ~= 0 then LocalPlayer.UserId = spoofConfig.AvatarUserId
        else
            LocalPlayer.UserId = origUserId
        end
    end)
end
end

-- ----------------------------------------------------------------------------
-- Function [99]: hookGetPlayerByUserId (374 bytes)
-- Description: Hooks Players:GetPlayerByUserId to resolve spoofed AvatarUserId and RealUserIdMap entries.
-- Resolved Upvalues: [1] = spoofConfig, [2] = localOriginalUserId, [3] = origGetPlayerByUserId, [4] = localSpoofTargetId
-- ----------------------------------------------------------------------------
do
    local spoofConfig, localOriginalUserId, origGetPlayerByUserId, localSpoofTargetId
    YabujinDeobfuscated.hookGetPlayerByUserId = function(b, J)
        if spoofConfig.AvatarSpoof and spoofConfig.AvatarUserId then
        if type(J) == "number" and(J == spoofConfig.AvatarUserId or J == localOriginalUserId) then
            return origGetPlayerByUserId(b, localSpoofTargetId)
        end
    end
    local N = origGetPlayerByUserId(b, J)
    if not N and type(J) == "number" and spoofConfig.RealUserIdMap then
        for A, q in pairs(spoofConfig.RealUserIdMap) do
            if q == J then
                local J = origGetPlayerByUserId(b, A)
                if J then
                    return J
                end
            end
        end
    end
    return N
end
end

-- ----------------------------------------------------------------------------
-- Function [1]: hookPlayerGetAttribute (1146 bytes)
-- Description: Hooks Player:GetAttribute to spoof Streak, ELO, LeaderboardRank, Level, and PlayerStatus ('Prime'/'Contraband') for yabujinUsers and AnonymousMode.
-- Resolved Upvalues: [1] = getRealUserId, [2] = getYabujinSpoofField, [3] = spoofConfig, [4] = origGetAttribute
-- ----------------------------------------------------------------------------
do
    local getRealUserId, getYabujinSpoofField, spoofConfig, origGetAttribute
    YabujinDeobfuscated.hookPlayerGetAttribute = function(b, J)
        if typeof(b) == "Instance" and(b:IsA("Player")) then local N = getRealUserId(b)
        if yabujinUsers[N] and yabujinUsers[N].spoofer then
            if J == "StatisticDuelsWinStreak" then
                local A = getYabujinSpoofField(N, "Streak")
                if A then
                    return A
                end
            elseif J == "DisplayELO" then
                local A = getYabujinSpoofField(N, "ELO")
                if A then
                    return A
                end
            elseif J == "DisplayELOLeaderboardRanking" then
                local A = getYabujinSpoofField(N, "LeaderboardRank")
                if A then
                    return A
                end
            elseif J == "Level" then
                local A = getYabujinSpoofField(N, "Level")
                if A then
                    return A
                end
            elseif J == "PlayerStatus" then
                local A = getYabujinSpoofField(N, "Pass")
                if A and type(A) == "number" and A > 1 then
                    if A == 2 then
                        return "Prime"
                    elseif A == 3 then
                        return "Contraband"
                    end
                end
            end
        end
    end
    if spoofConfig.AnonymousMode then
        local N = getRealUserId(b)
        if not(yabujinUsers[N] and yabujinUsers[N].spoofer) then
            if J == "StatisticDuelsWinStreak" and spoofConfig.AnonStreak then
                return spoofConfig.AnonStreak
            end
            if J == "DisplayELO" and spoofConfig.AnonELO then
                return spoofConfig.AnonELO
            end
            if J == "Level" and spoofConfig.AnonLevel then
                return spoofConfig.AnonLevel
            end
            if J == "DisplayELOLeaderboardRanking" then
                return 0
            end
        end
    end
    return origGetAttribute(b, J)
end
end

-- ----------------------------------------------------------------------------
-- Function [8]: hookNametagDisplayName (527 bytes)
-- Description: Hooks player nametag/chat display name for yabujinUsers, AnonymousMode, and local UsernameSpoof.
-- Resolved Upvalues: [1] = LocalPlayer, [2] = getRealUserId, [3] = getSpoofedPlayerNames, [4] = spoofConfig, [5] = origNametagFunc
-- ----------------------------------------------------------------------------
do
    local LocalPlayer, getRealUserId, getSpoofedPlayerNames, spoofConfig, origNametagFunc
    YabujinDeobfuscated.hookNametagDisplayName = function(b, J, N, A)
        if J and J ~= LocalPlayer then local q = getRealUserId(J)
        local W = yabujinUsers[q] or yabujinUsers[tostring(q)]
        if W and W.spoofer and(next(W.spoofer)) then
            local q, q = getSpoofedPlayerNames(J)
            N = if q and q ~= "" then q else N
        elseif spoofConfig.AnonymousMode then
            return origNametagFunc(b, J, spoofConfig.AnonDisplayName or "Roblox", A)
        end
    end
    return origNametagFunc(b, J, if J == LocalPlayer and spoofConfig.UsernameSpoof then spoofConfig.SpoofedDisplayName ~= "" and spoofConfig.SpoofedDisplayName or(spoofConfig.SpoofedName ~= "" and spoofConfig.SpoofedName or J.DisplayName) else N, A)
end
end

-- ----------------------------------------------------------------------------
-- Function [20]: hookClientDataGet (9874 bytes)
-- Description: Core Rivals client data interceptor: spoofs Seasons, Ranked ELO, LeaderboardRank, BattlePass, Currencies (Keys, UnlockTokens, EventCurrency, Glory, SkinTickets), CosmeticInventory, WeaponInventory, EquippedCosmetics, EquippedEmotes, Statistics, FavoriteWeapons/Map, WeaponStatistics, DuelHistory, and LoggedELOEvents.
-- Resolved Upvalues: [1] = spooferActive, [2] = origDataGet, [3] = spoofConfig, [4] = rankedConfig, [5] = cachedSeasonsTable, [6] = cachedOrigSeasons, [7] = cachedSeasonsKey, [8] = equippedCosmeticsOverride, [9] = isCosmeticUnlocked, [10] = weaponUnlockMode, [11] = cachedWeaponInventory, [12] = specificUnlockedWeapons, [13] = ItemDatabase, [14] = favoritedWeaponsMap, [15] = WeaponConfig, [16] = favoritedCosmeticsOverride, [17] = equippedEmotesOverride, [18] = streakSpoofState
-- ----------------------------------------------------------------------------
do
    local spooferActive, origDataGet, spoofConfig, rankedConfig, cachedSeasonsTable, cachedOrigSeasons, cachedSeasonsKey, equippedCosmeticsOverride, isCosmeticUnlocked, weaponUnlockMode, cachedWeaponInventory, specificUnlockedWeapons, ItemDatabase, favoritedWeaponsMap, WeaponConfig, favoritedCosmeticsOverride, equippedEmotesOverride, streakSpoofState
    YabujinDeobfuscated.hookClientDataGet = function(b, J, ...)
        if not spooferActive then return origDataGet(b, J, ...)
    end
    if spoofConfig.HideBeginnerTasks and J == "BeginnerTasksCompleted" then
        return math.huge
    end
    if J == "Seasons" then
        local N = rankedConfig.forceRankedFakeElo and spoofConfig._forceRankedSummaryElo ~= nil
        if not spoofConfig.ELOSpoof and not spoofConfig.PassSpoof and not spoofConfig.PassLevelSpoof and not N then
            return origDataGet(b, J, ...)
        end
        local A = origDataGet(b, J, ...)
        local q = tostring(spoofConfig.ELOSpoof) .. "|" .. tostring(spoofConfig.SpoofedELO or "") .. "|" .. tostring(spoofConfig.SpoofedLeaderboardRank or "") .. "|" .. tostring(rankedConfig.forceRankedFakeElo) .. "|" .. tostring(spoofConfig._forceRankedSummaryElo or "")
        if cachedSeasonsTable and cachedOrigSeasons == A and cachedSeasonsKey == q then
            return cachedSeasonsTable
        end
        if A then
            local W = table.clone(A)
            for U, L in pairs(W) do
                local h, B, R = false, if N then spoofConfig._forceRankedSummaryElo else
                if spoofConfig.ELOSpoof and spoofConfig.SpoofedELO then
                    spoofConfig.SpoofedELO
                else
                    nil
                    if B and L.RankedPerformances then
                        R = table.clone(L)
                        local s = table.clone(L.RankedPerformances)
                        for d, w in pairs(s) do
                            local u = table.clone(w)
                            u.CurrentELO = B
                            if not N and spoofConfig.SpoofedLeaderboardRank then
                                u.CurrentLeaderboardRanking = spoofConfig.SpoofedLeaderboardRank
                                if u.Metadata then
                                    w = table.clone(u.Metadata)
                                    w.FinalLeaderboardRank = spoofConfig.SpoofedLeaderboardRank
                                    u.Metadata = w
                                end
                            elseif not N then
                                u.CurrentLeaderboardRanking = nil
                            end
                            if u.HighestELO and u.HighestELO < B then
                                u.HighestELO = B
                            end
                            s[d] = u
                        end
                        R.RankedPerformances = s
                        h = true
                    else
                        R = L
                    end
                    if(spoofConfig.PassSpoof and spoofConfig.SpoofPassTrack or spoofConfig.PassLevelSpoof and spoofConfig.SpoofedPassLevel) and L.BattlePass then
                        R = if R == L then(table.clone(L)) else R
                        B = table.clone(L.BattlePass)
                        if spoofConfig.PassSpoof and spoofConfig.SpoofPassTrack then
                            if spoofConfig.SpoofPassTrack == 3 then
                                B.MaxPassTrackNum = 2
                                B.SeasonPassBundleOwned = true
                            else
                                B.MaxPassTrackNum = spoofConfig.SpoofPassTrack
                                B.SeasonPassBundleOwned = false
                            end
                        end
                        if spoofConfig.PassLevelSpoof and spoofConfig.SpoofedPassLevel then
                            B.PassLevel = spoofConfig.SpoofedPassLevel
                        end
                        R.BattlePass = B
                        h = true
                    end
                    if h then
                        W[U] = R
                    end
                end
                cachedSeasonsTable, cachedOrigSeasons, cachedSeasonsKey = W, A, q
                return W
            end
            return A
        end
        if J == "EquippedCosmetics" then
            local N = origDataGet(b, J, ...)
            if not next(equippedCosmeticsOverride) then
                return N
            end
            local A = N and(table.clone(N)) or{}
            for N, q in pairs(equippedCosmeticsOverride) do
                A[N] = A[N] or{}
                for W, U in pairs(q) do
                    A[N][W] = U
                end
            end
            return A
        end
        if J == "Level" then
            if spoofConfig.LevelSpoof and spoofConfig.SpoofedLevel then
                return spoofConfig.SpoofedLevel
            end
            return origDataGet(b, J, ...)
        end
        if J == "WeaponKeys" then
            if spoofConfig.KeysSpoof and spoofConfig.SpoofedKeys then
                return spoofConfig.SpoofedKeys
            end
            return origDataGet(b, J, ...)
        end
        if J == "UnlockTokens" then
            if spoofConfig.TokensSpoof and spoofConfig.SpoofedTokens then
                return spoofConfig.SpoofedTokens
            end
            return origDataGet(b, J, ...)
        end
        if J == "EventCurrency" then
            if spoofConfig.EventCurrSpoof and spoofConfig.SpoofedEventCurr then
                return spoofConfig.SpoofedEventCurr
            end
            return origDataGet(b, J, ...)
        end
        if J == "Glory" then
            if spoofConfig.GlorySpoof and spoofConfig.SpoofedGlory then
                return spoofConfig.SpoofedGlory
            end
            return origDataGet(b, J, ...)
        end
        if J == "SkinTickets" then
            if spoofConfig.SkinTicketsSpoof and spoofConfig.SpoofedSkinTickets then
                return spoofConfig.SpoofedSkinTickets
            end
            return origDataGet(b, J, ...)
        end
        if J == "StatisticPlaytime" then
            if spoofConfig.PlaytimeSpoof and spoofConfig.SpoofedPlaytime then
                return spoofConfig.SpoofedPlaytime
            end
            return origDataGet(b, J, ...)
        end
        if J == "CosmeticInventory" then
            if spoofConfig._cachedInvProxy and spoofConfig._cachedInvProxySelf == b then
                return spoofConfig._cachedInvProxy
            end
            local N, A, q = b, 0
            local W = setmetatable({}, {__index = function(U, L)
                if not spooferActive then
                if q then
                    return rawget(q, L)
                end
                return nil
            end
            U = tick()
            if not q or U -A > 2 then
                q, A = origDataGet(N, "CosmeticInventory"), U
            end
            if q and(rawget(q, L)) then
                return rawget(q, L)
            end
            if isCosmeticUnlocked(L) then
                return true
            end
        end
        })
        spoofConfig._cachedInvProxy = W
        spoofConfig._cachedInvProxySelf = b
        return W
    end
    if J == "WeaponInventory" then
        if weaponUnlockMode == "none" then
            local N = origDataGet(b, J, ...)
            if type(spoofConfig._scriptUnlockedWeapons) ~= "table" or not next(spoofConfig._scriptUnlockedWeapons) then
                return N
            end
            local A = {}
            if N then
                for q, W in pairs(N) do
                    if not spoofConfig._scriptUnlockedWeapons[W and W.Name or q] then
                        A[q] = W
                    end
                end
            end
            return A
        end
        if cachedWeaponInventory then
            return cachedWeaponInventory
        end
        local N = origDataGet(b, J, ...)
        local A = {}
        if weaponUnlockMode == "specific" then
            if N then
                for q, W in pairs(N) do
                    if specificUnlockedWeapons[q] then
                        A[q] = W
                    end
                end
            end
            for q in pairs(specificUnlockedWeapons) do
                if not A[q] then
                    local W = ItemDatabase.Items[q]
                    if W and W.Class then
                        A[q] = {Name = q, Owned = true, Class = W.Class, Unlocked = true, Level = 1, XP = 0, IsFavorited = favoritedWeaponsMap[q] or false}
                    end
                end
            end
        elseif weaponUnlockMode == "unlockable" then
            if N then
                for q, W in pairs(N) do
                    A[q] = W
                end
            end
            for q, W in ipairs(WeaponConfig.OwnableWeapons or{}) do
                if not A[W] then
                    q = ItemDatabase.Items[W]
                    if q and q.Class then
                        A[W] = {Name = W, Owned = true, Class = q.Class, Unlocked = true, Level = 1, XP = 0, IsFavorited = favoritedWeaponsMap[W] or false}
                    end
                end
            end
        elseif weaponUnlockMode == "all" then
            if N then
                for q, W in pairs(N) do
                    A[q] = W
                end
            end
            for N, q in pairs(ItemDatabase.Items) do
                if not A[N] and q and q.Class then
                    A[N] = {Name = N, Owned = true, Class = q.Class, Unlocked = true, Level = 1, XP = 0, IsFavorited = favoritedWeaponsMap[N] or false}
                end
            end
        end
        for N, q in pairs(A) do
            if favoritedWeaponsMap[N] ~= nil then
                q.IsFavorited = favoritedWeaponsMap[N]
            end
        end
        cachedWeaponInventory = A
        return A
    end
    if J == "FavoritedCosmetics" then
        local N = origDataGet(b, J, ...)
        if not next(favoritedCosmeticsOverride) then
            return N
        end
        local A, q = N and(table.clone(N)) or{}, origDataGet(b, "CosmeticInventory")
        for N, W in pairs(favoritedCosmeticsOverride) do
            for U, L in pairs(W) do
                if L and(isCosmeticUnlocked(U) or q and(rawget(q, U))) then
                    A[N] = A[N] or{}
                    A[N][U] = L
                end
            end
        end
        return A
    end
    if J == "EquippedEmotes" then
        if not next(equippedEmotesOverride) then
            return origDataGet(b, J, ...)
        end
        local N = origDataGet(b, J, ...)
        local A = N and(table.clone(N)) or{}
        for N, q in pairs(equippedEmotesOverride) do
            if q and q.Name and q.Name ~= "" then
                A[N] = q
            end
        end
        return A
    end
    if J == "RankedELO" or J == "RankedCurrentELO" or J == "ELO" then
        if spoofConfig.ELOSpoof and spoofConfig.SpoofedELO then
            return spoofConfig.SpoofedELO
        end
        return origDataGet(b, J, ...)
    end
    if J == "Statistics" then
        if not spoofConfig.StatsSpoof or not spoofConfig.SpoofedStats then
            return origDataGet(b, J, ...)
        end
        local N = origDataGet(b, J, ...)
        if not N then
            return N
        end
        local A, q = table.clone(N), spoofConfig.SpoofedStats
        if q.casualWins then
            A.CasualWins = q.casualWins
        end
        if q.casualLosses then
            A.CasualLosses = q.casualLosses
        end
        if q.rankedWins then
            A.RankedWins = q.rankedWins
        end
        if q.rankedLosses then
            A.RankedLosses = q.rankedLosses
        end
        if q.streak then
            A.Streak = q.streak
        end
        return A
    end
    if spoofConfig.StatsSpoof and spoofConfig.SpoofedStats then
        local N = spoofConfig.SpoofedStats
        if J == "StatisticDuelsWon" and N.casualWins then
            return N.casualWins
        end
        if J == "StatisticDuelsLost" and N.casualLosses then
            return N.casualLosses
        end
        if J == "StatisticDuelsWinStreak" and streakSpoofState.streakEnabled then
            local A = streakSpoofState.spoofedStreak or 0
            if streakSpoofState.streakStackReal then
                return(origDataGet(b, J, ...) or 0) + A
            end
            return A
        end
        if J == "StatisticRankedDuelsWon" and N.rankedWins then
            return N.rankedWins
        end
        if J == "StatisticRankedDuelsLost" and N.rankedLosses then
            return N.rankedLosses
        end
        if J == "StatisticDuelsWinPercent" then
            local A = N.casualWins or(origDataGet(b, "StatisticDuelsWon")) or 0
            local q = A + (N.casualLosses or(origDataGet(b, "StatisticDuelsLost")) or 0)
            if q > 0 then
                return A / q
            end
        end
        if J == "StatisticRankedDuelsWinPercent" then
            local A = N.rankedWins or(origDataGet(b, "StatisticRankedDuelsWon")) or 0
            local q = A + (N.rankedLosses or(origDataGet(b, "StatisticRankedDuelsLost")) or 0)
            if q > 0 then
                return A / q
            end
        end
        if J == "StatisticDuelsPlayed" then
            local A, q = N.casualWins or(origDataGet(b, "StatisticDuelsWon")) or 0, N.casualLosses or(origDataGet(b, "StatisticDuelsLost")) or 0
            if N.casualWins or N.casualLosses then
                return A + q
            end
        end
        if J == "StatisticRankedDuelsPlayed" then
            local A, q = N.rankedWins or(origDataGet(b, "StatisticRankedDuelsWon")) or 0, N.rankedLosses or(origDataGet(b, "StatisticRankedDuelsLost")) or 0
            if N.rankedWins or N.rankedLosses then
                return A + q
            end
        end
    end
    if spoofConfig.FavSpoof then
        if J == "StatisticFavoriteWeaponPrimary" and spoofConfig.SpoofedFavWeapons and spoofConfig.SpoofedFavWeapons.Primary then
            return spoofConfig.SpoofedFavWeapons.Primary
        end
        if J == "StatisticFavoriteWeaponSecondary" and spoofConfig.SpoofedFavWeapons and spoofConfig.SpoofedFavWeapons.Secondary then
            return spoofConfig.SpoofedFavWeapons.Secondary
        end
        if J == "StatisticFavoriteWeaponMelee" and spoofConfig.SpoofedFavWeapons and spoofConfig.SpoofedFavWeapons.Melee then
            return spoofConfig.SpoofedFavWeapons.Melee
        end
        if J == "StatisticFavoriteWeaponUtility" and spoofConfig.SpoofedFavWeapons and spoofConfig.SpoofedFavWeapons.Utility then
            return spoofConfig.SpoofedFavWeapons.Utility
        end
        if J == "StatisticFavoriteWeapon" and spoofConfig.SpoofedFavWeapons then
            local N = spoofConfig.SpoofedFavWeapons.Primary or spoofConfig.SpoofedFavWeapons.Secondary or spoofConfig.SpoofedFavWeapons.Melee or spoofConfig.SpoofedFavWeapons.Utility
            if N then
                return N
            end
        end
        if J == "StatisticFavoriteMap" and spoofConfig.SpoofedFavMap then
            return spoofConfig.SpoofedFavMap
        end
    end
    if J == "WeaponStatistics" then
        local N, A = origDataGet(b, J, ...), spoofConfig.ContractSpoof
        if A and A.enabled and A.perWeapon then
            local q = N and(table.clone(N)) or{}
            for W, U in pairs(A.perWeapon) do
                if U and(U.kills or U.playtime) then
                    local L = q[W] and(table.clone(q[W])) or{}
                    if A.stackReal then
                        if U.kills then
                            L.StatisticEliminations = (L.StatisticEliminations or 0) + U.kills
                        end
                        if U.playtime then
                            L.StatisticPlaytime = (L.StatisticPlaytime or 0) + U.playtime
                        end
                    else
                        if U.kills then
                            L.StatisticEliminations = U.kills
                        end
                        if U.playtime then
                            L.StatisticPlaytime = U.playtime
                        end
                    end
                    q[W] = L
                end
            end
            return q
        end
        return N
    end
    if J == "DuelHistory" then
        local N = origDataGet(b, J, ...)
        if not spoofConfig.DuelHistorySpoof or not spoofConfig.SpoofedDuelHistory or#spoofConfig.SpoofedDuelHistory == 0 then
            return N
        end
        local A = {}
        for q, q in ipairs(spoofConfig.SpoofedDuelHistory) do
            A[#A + 1] = q
        end
        if spoofConfig.DuelHistoryReplace then
            return A
        end
        if N and type(N) == "table" then
            for q, q in ipairs(N) do
                A[#A + 1] = q
            end
        end
        return A
    end
    if J == "LoggedELOEvents" then
        local N = origDataGet(b, J, ...)
        if not spoofConfig.DuelHistorySpoof or not spoofConfig.SpoofedELOEvents or#spoofConfig.SpoofedELOEvents == 0 then
            return N
        end
        local A = {}
        for q, q in ipairs(spoofConfig.SpoofedELOEvents) do
            A[#A + 1] = q
        end
        if not spoofConfig.DuelHistoryReplace and N and type(N) == "table" then
            for q, q in ipairs(N) do
                A[#A + 1] = q
            end
        end
        return A
    end
    return origDataGet(b, J, ...)
end
end

-- ----------------------------------------------------------------------------
-- Function [126]: isCosmeticUnlockedCached (320 bytes)
-- Description: Checks whether a cosmetic item is unlocked across 5 cosmetic tables with a cfg.cacheTtl timestamp cache.
-- Resolved Upvalues: [1] = forceUnlockAll, [2] = unlockerEnabled, [3] = ownershipCache, [4] = timestampCache, [5] = unlockedTable1, [6] = unlockedTable2, [7] = unlockedTable3, [8] = unlockedTable4, [9] = unlockedTable5
-- ----------------------------------------------------------------------------
do
    local forceUnlockAll, unlockerEnabled, ownershipCache, timestampCache, unlockedTable1, unlockedTable2, unlockedTable3, unlockedTable4, unlockedTable5
    YabujinDeobfuscated.isCosmeticUnlockedCached = function(b)
        if not forceUnlockAll and not unlockerEnabled then return false
    end
    local J = tick()
    local N = ownershipCache[b]
    if N ~= nil and J -(timestampCache[b] or 0) < cfg.cacheTtl then
        return N
    end
    N = unlockedTable1[b] or unlockedTable2[b] or unlockedTable3[b] or unlockedTable4[b] or unlockedTable5[b]
    ownershipCache[b] = N
    timestampCache[b] = J
    return N
end
end

-- ----------------------------------------------------------------------------
-- Function [18]: hookOwnsWeaponCosmetic (542 bytes)
-- Description: Hooks weapon cosmetic ownership check (ownsWep) to unlock skins/cosmetics.
-- Resolved Upvalues: [1] = unlockerEnabled, [2] = origCosmeticController, [3] = spoofConfig, [4] = unlockedTable1, [5] = unlockedTable2, [6] = unlockedTable3, [7] = unlockedTable4, [8] = unlockedTable5, [9] = weaponSkinsMap1, [10] = weaponSkinsMap2, [11] = weaponSkinsMap3
-- ----------------------------------------------------------------------------
do
    local unlockerEnabled, origCosmeticController, spoofConfig, unlockedTable1, unlockedTable2, unlockedTable3, unlockedTable4, unlockedTable5, weaponSkinsMap1, weaponSkinsMap2, weaponSkinsMap3
    YabujinDeobfuscated.hookOwnsWeaponCosmetic = function(b, J, N, A)
        if not N or string.sub(N, 1, 8) == "MISSING_" then return false
    end
    if not unlockerEnabled then
        return origCosmeticController.ownsWep(b, J, N, A)
    end
    if spoofConfig._menuLockedCosmetics and spoofConfig._menuLockedCosmetics[N] then
        return false
    end
    if origCosmeticController.ownsWep(b, J, N, A) then
        return true
    end
    if unlockedTable1[N] or unlockedTable2[N] or unlockedTable3[N] or unlockedTable4[N] or unlockedTable5[N] then
        return true
    end
    if A then
        J = weaponSkinsMap1[N]
        if J and J[A] then
            return true
        end
        b = weaponSkinsMap2[N]
        if b and b[A] then
            return true
        end
        local b = weaponSkinsMap3[N]
        if b and b[A] then
            return true
        end
    end
    return false
end
end

-- ----------------------------------------------------------------------------
-- Function [63]: hookOwnsUniversalCosmetic (357 bytes)
-- Description: Hooks universal cosmetic ownership check (ownsUniv).
-- Resolved Upvalues: [1] = unlockerEnabled, [2] = origCosmeticController, [3] = spoofConfig, [4] = unlockedTable1, [5] = unlockedTable2, [6] = unlockedTable3, [7] = unlockedTable4, [8] = unlockedTable5
-- ----------------------------------------------------------------------------
do
    local unlockerEnabled, origCosmeticController, spoofConfig, unlockedTable1, unlockedTable2, unlockedTable3, unlockedTable4, unlockedTable5
    YabujinDeobfuscated.hookOwnsUniversalCosmetic = function(b, J, N)
        if not N or string.sub(N, 1, 8) == "MISSING_" then return false
    end
    if not unlockerEnabled then
        return origCosmeticController.ownsUniv(b, J, N)
    end
    if spoofConfig._menuLockedCosmetics and spoofConfig._menuLockedCosmetics[N] then
        return false
    end
    if origCosmeticController.ownsUniv(b, J, N) then
        return true
    end
    return unlockedTable1[N] or unlockedTable2[N] or unlockedTable3[N] or unlockedTable4[N] or unlockedTable5[N] or false
end
end

-- ----------------------------------------------------------------------------
-- Function [69]: hookOwnsGeneralCosmetic (532 bytes)
-- Description: Hooks general cosmetic ownership check (owns).
-- Resolved Upvalues: [1] = unlockerEnabled, [2] = origCosmeticController, [3] = spoofConfig, [4] = unlockedTable1, [5] = unlockedTable2, [6] = unlockedTable3, [7] = unlockedTable4, [8] = unlockedTable5, [9] = weaponSkinsMap1, [10] = weaponSkinsMap2, [11] = weaponSkinsMap3
-- ----------------------------------------------------------------------------
do
    local unlockerEnabled, origCosmeticController, spoofConfig, unlockedTable1, unlockedTable2, unlockedTable3, unlockedTable4, unlockedTable5, weaponSkinsMap1, weaponSkinsMap2, weaponSkinsMap3
    YabujinDeobfuscated.hookOwnsGeneralCosmetic = function(b, J, N, A)
        if not N or string.sub(N, 1, 8) == "MISSING_" then return false
    end
    if not unlockerEnabled then
        return origCosmeticController.owns(b, J, N, A)
    end
    if spoofConfig._menuLockedCosmetics and spoofConfig._menuLockedCosmetics[N] then
        return false
    end
    if origCosmeticController.owns(b, J, N, A) then
        return true
    end
    if unlockedTable1[N] or unlockedTable2[N] or unlockedTable3[N] or unlockedTable4[N] or unlockedTable5[N] then
        return true
    end
    if A then
        J = weaponSkinsMap1[N]
        if J and J[A] then
            return true
        end
        b = weaponSkinsMap2[N]
        if b and b[A] then
            return true
        end
        local b = weaponSkinsMap3[N]
        if b and b[A] then
            return true
        end
    end
    return false
end
end

-- ----------------------------------------------------------------------------
-- Function [73]: hookOwnsCategoryCosmetic (494 bytes)
-- Description: Hooks category cosmetic ownership check (ownsSome).
-- Resolved Upvalues: [1] = unlockerEnabled, [2] = origCosmeticController, [3] = spoofConfig, [4] = isCosmeticUnlocked, [5] = weaponSkinsMap1, [6] = weaponSkinsMap2, [7] = weaponSkinsMap3
-- ----------------------------------------------------------------------------
do
    local unlockerEnabled, origCosmeticController, spoofConfig, isCosmeticUnlocked, weaponSkinsMap1, weaponSkinsMap2, weaponSkinsMap3
    YabujinDeobfuscated.hookOwnsCategoryCosmetic = function(b, J, N)
        if not N or string.sub(N, 1, 8) == "MISSING_" then return false
    end
    if not unlockerEnabled then
        return origCosmeticController.ownsSome(b, J, N)
    end
    if spoofConfig._menuLockedCosmetics and spoofConfig._menuLockedCosmetics[N] then
        return false
    end
    if origCosmeticController.ownsSome(b, J, N) then
        return true
    end
    if isCosmeticUnlocked(N) then
        return true
    end
    J = weaponSkinsMap1[N]
    if J and(next(J)) then
        return true
    end
    b = weaponSkinsMap2[N]
    if b and(next(b)) then
        return true
    end
    b = weaponSkinsMap3[N]
    if b and(next(b)) then
        return true
    end
    return false
end
end

-- ----------------------------------------------------------------------------
-- Function [96]: hookOwnsNormalCosmetic (371 bytes)
-- Description: Hooks normal cosmetic ownership check (ownsNormal).
-- Resolved Upvalues: [1] = unlockerEnabled, [2] = origCosmeticController, [3] = spoofConfig, [4] = unlockedTable1, [5] = unlockedTable2, [6] = unlockedTable3, [7] = unlockedTable4, [8] = unlockedTable5
-- ----------------------------------------------------------------------------
do
    local unlockerEnabled, origCosmeticController, spoofConfig, unlockedTable1, unlockedTable2, unlockedTable3, unlockedTable4, unlockedTable5
    YabujinDeobfuscated.hookOwnsNormalCosmetic = function(b, J, N, A)
        if not N or string.sub(N, 1, 8) == "MISSING_" then return false
    end
    if not unlockerEnabled then
        return origCosmeticController.ownsNormal(b, J, N, A)
    end
    if spoofConfig._menuLockedCosmetics and spoofConfig._menuLockedCosmetics[N] then
        return false
    end
    if origCosmeticController.ownsNormal(b, J, N, A) then
        return true
    end
    return unlockedTable1[N] or unlockedTable2[N] or unlockedTable3[N] or unlockedTable4[N] or unlockedTable5[N] or false
end
end

-- ----------------------------------------------------------------------------
-- Function [102]: hookShopPurchaseSkinTickets (596 bytes)
-- Description: Hooks shop purchase controller to spoof SkinTickets purchases, deduct SpoofedSkinTickets, replicate CurrentData, and trigger _onSkinTicketPurchase.
-- Resolved Upvalues: [1] = spoofConfig, [2] = savedSettings, [3] = pcall, [4] = clientDataController, [5] = origShopPurchase
-- ----------------------------------------------------------------------------
do
    local spoofConfig, savedSettings, pcall, clientDataController, origShopPurchase
    YabujinDeobfuscated.hookShopPurchaseSkinTickets = function(b, J, N, ...)
        if spoofConfig.SkinTicketsSpoof and N == "SkinTickets" then local A = b
        task.spawn(function()
            game:GetService("RunService").Heartbeat:Wait()
            local q = A:GetShopEntry(J)
            if q then
                local A = q.Prices and q.Prices[N] or 10
                if spoofConfig.SpoofedSkinTickets then
                    spoofConfig.SpoofedSkinTickets = math.max(0, spoofConfig.SpoofedSkinTickets -A)
                    savedSettings.spoofSkinTickets = spoofConfig.SpoofedSkinTickets
                    pcall(function()
                        clientDataController.CurrentData:Replicate("SkinTickets")
                    end)
                end
                if spoofConfig._onSkinTicketPurchase then
                    spoofConfig._onSkinTicketPurchase(q)
                end
            end
        end)
        return
    end
    return origShopPurchase(b, J, N, ...)
end
end

-- ----------------------------------------------------------------------------
-- Function [48]: hookLeaderboardSurfaceGui (1136 bytes)
-- Description: Hooks in-world Leaderboard SurfaceGui slot generation to spoof avatar headshots for LocalPlayer and yabujinUsers.
-- Resolved Upvalues: [1] = pcall, [2] = origGenerateLeaderboard, [3] = localUserId, [4] = spoofConfig, [5] = leaderboardConstants, [6] = getYabujinSpoofField
-- ----------------------------------------------------------------------------
do
    local pcall, origGenerateLeaderboard, localUserId, spoofConfig, leaderboardConstants, getYabujinSpoofField
    YabujinDeobfuscated.hookLeaderboardSurfaceGui = function(b)
        pcall(function()
            for J, J in pairs(b._player_slots or{}) do
                pcall(function()
                    J:Destroy()
                end)
            end
        end)
        b._player_slots = {}
        b._generate_hash = (b._generate_hash or 0) + 1
        if b.SurfaceGui and b.SurfaceGui.Title then
            b.SurfaceGui.Title.Text = "Loading..."
        end
        local J = origGenerateLeaderboard(b)
        pcall(function()
            local N = b._leaderboards_serial and b._leaderboards_serial[b._leaderboard_name]
            local A = N and N.Players
            if not A or#A == 0 or not b._player_slots then
                return
            end
            for q, W in ipairs(b._player_slots) do
                N = A[q]
                if not N then
                    break
                end
                q = tonumber(N.key)
                if not q then
                    break
                end
                if q == localUserId or spoofConfig.AvatarSpoof and spoofConfig.AvatarUserId and q == spoofConfig.AvatarUserId then
                    if spoofConfig.AvatarSpoof and spoofConfig.AvatarUserId and spoofConfig.AvatarUserId ~= 0 then
                        pcall(function()
                            W.Left.Headshot.Image = string.format(leaderboardConstants.HEADSHOT_IMAGE, spoofConfig.AvatarUserId)
                        end)
                    end
                elseif yabujinUsers[q] and yabujinUsers[q].spoofer then
                    local b = getYabujinSpoofField(q, "Avatar")
                    if b and type(b) == "number" and b ~= 0 then
                        pcall(function()
                            W.Left.Headshot.Image = string.format(leaderboardConstants.HEADSHOT_IMAGE, b)
                        end)
                    end
                end
            end
        end)
        return J
    end
end

-- ----------------------------------------------------------------------------
-- Function [13]: hookGetNetworkPing (524 bytes)
-- Description: Hooks network ping getter to spoof ping (Static, Random, Multiply, Divide, Plus, Minus).
-- Resolved Upvalues: [1] = origGetPing, [2] = pingConfig
-- ----------------------------------------------------------------------------
do
    local origGetPing, pingConfig
    YabujinDeobfuscated.hookGetNetworkPing = function(b, J, ...)
        local N = origGetPing(b, J, ...)
        if pingConfig.PingSpoof then
            J, b = pingConfig.PingMode or "Static", pingConfig.PingValue or 20
            if J == "Static" then
                return b
            elseif J == "Random" then
                local A, q = tonumber(pingConfig.PingRandomMin) or 1, tonumber(pingConfig.PingRandomMax) or 100
                if A > q then
                    q, A = A, q
                end
                return math.random(A, q)
            elseif J == "Multiply" then
                return math.floor(N * b)
            elseif J == "Divide" then
                return math.floor(N / b)
            elseif J == "Plus" then
                return math.floor(N + b)
            elseif J == "Minus" then
                return math.floor(N -b)
            end
        end
        return N
    end
end

-- ----------------------------------------------------------------------------
-- Function [60]: updateStretchedResolution (809 bytes)
-- Description: Applies horizontal/vertical aspect-ratio stretch to CurrentCamera.CFrame and compensates BillboardGui StudsOffsetWorldSpace across all player characters.
-- Resolved Upvalues: [1] = Workspace, [2] = stretchConfig, [3] = Players, [4] = origBillboardOffsets
-- ----------------------------------------------------------------------------
do
    local Workspace, stretchConfig, Players, origBillboardOffsets
    YabujinDeobfuscated.updateStretchedResolution = function()
        local b = Workspace.CurrentCamera
        if not b then
            return
        end
        local J, N = stretchConfig.x, stretchConfig.y
        if J ~= 1 and J > 0 then
            b.CFrame = b.CFrame * CFrame.new(0, 0, 0, J, 0, 0, 0, 1, 0, 0, 0, 1)
            local A = b.CFrame.Position
            local q = b.CFrame.RightVector
            local W = 1 / J -1
            for J, U in ipairs(Players:GetPlayers()) do
                J = U.Character
                if J then
                    for U, U in ipairs(J:GetChildren()) do
                        if U:IsA("BasePart") then
                            for J, L in ipairs(U:GetChildren()) do
                                if L:IsA("BillboardGui") then
                                    if not origBillboardOffsets[L] then
                                        origBillboardOffsets[L] = L.StudsOffsetWorldSpace
                                    end
                                    J = ((L.Adornee and typeof(L.Adornee) == "Instance" and(L.Adornee:IsA("BasePart")) and L.Adornee.Position or U.Position) -A):Dot(q)
                                    L.StudsOffsetWorldSpace = origBillboardOffsets[L] + q * (J * W)
                                end
                            end
                        end
                    end
                end
            end
        end
        if N ~= 1 then
            b.CFrame = b.CFrame * CFrame.new(0, 0, 0, 1, 0, 0, 0, N, 0, 0, 0, 1)
        end
    end
end

-- ----------------------------------------------------------------------------
-- Function [17]: updateCameraMotionBlur (418 bytes)
-- Description: Updates camera motion blur Size each frame based on camera velocity and angular rotation.
-- Resolved Upvalues: [1] = motionBlurState
-- ----------------------------------------------------------------------------
do
    local motionBlurState
    YabujinDeobfuscated.updateCameraMotionBlur = function(b)
        local J = workspace.CurrentCamera
        if not J or not motionBlurState.blur then
            return
        end
        local N = J.CFrame
        if motionBlurState.lastCFrame then
            J = motionBlurState.lastCFrame
            local A, q = (N.Position -J.Position).Magnitude / math.max(b, 0.004166666666666667), (N.LookVector -J.LookVector).Magnitude * 60
            local b = math.clamp(A * 0.05 + q * motionBlurState.strength * 0.5, 0, motionBlurState.strength)
            motionBlurState.blur.Size = motionBlurState.blur.Size * 0.5 + b * 0.5
        end
        motionBlurState.lastCFrame = N
    end
end

-- ----------------------------------------------------------------------------
-- Function [78]: syncPartToCameraCFrame (160 bytes)
-- Description: Synchronizes a world part's CFrame directly to workspace.CurrentCamera.CFrame.
-- Resolved Upvalues: [1] = targetPart, [2] = lastCameraCFrame
-- ----------------------------------------------------------------------------
do
    local targetPart, lastCameraCFrame
    YabujinDeobfuscated.syncPartToCameraCFrame = function()
        local b = workspace.CurrentCamera
        if b and targetPart.Parent then
            lastCameraCFrame = b.CFrame
            targetPart.CFrame = lastCameraCFrame
        end
    end
end

-- ----------------------------------------------------------------------------
-- Function [85]: syncPartAboveCamera (195 bytes)
-- Description: Positions a target part 32 studs above workspace.CurrentCamera.CFrame.Position.
-- Resolved Upvalues: [1] = pcall, [2] = targetPart
-- ----------------------------------------------------------------------------
do
    local pcall, targetPart
    YabujinDeobfuscated.syncPartAboveCamera = function()
        pcall(function()
            local b = workspace.CurrentCamera
            if b and targetPart.Parent then
                targetPart.CFrame = CFrame.new(b.CFrame.Position + Vector3.new(0, 32, 0))
            end
        end)
    end
end

-- ----------------------------------------------------------------------------
-- Function [107]: updateBezierPulseBeams (2106 bytes)
-- Description: Updates 3D cubic Bezier pulse beam parts (size, CFrame orientation, transparency, and ColorSequence keypoints).
-- Resolved Upvalues: [1] = activeBeamsList, [2] = heartbeatConnection, [3] = clockFunc, [4] = calcPulseAlpha, [5] = evalCubicBezier, [6] = simplexNoise, [7] = beamRotationOffset
-- ----------------------------------------------------------------------------
do
    local activeBeamsList, heartbeatConnection, clockFunc, calcPulseAlpha, evalCubicBezier, simplexNoise, beamRotationOffset
    YabujinDeobfuscated.updateBezierPulseBeams = function()
        if next(activeBeamsList) == nil then
        if heartbeatConnection then
            heartbeatConnection:Disconnect()
            heartbeatConnection = nil
        end
        return
    end
    for b, J in pairs(activeBeamsList) do
        if J.Enabled == true then
            J.PartsHidden = false
            local N, A, q, W, U, L = 1 -J.MaxTransparency, 1 -J.MinTransparency, J.MinRadius, J.MaxRadius, J.Thickness, J.Parts
            local h, B, R, s, d, w, u, C, o, H, z = #L, J.RanNum, J.StartT, J.AnimationSpeed, J.Frequency, J.MinThicknessMultiplier, J.MaxThicknessMultiplier, J.Attachment0, J.Attachment1, J.CurveSize0, J.CurveSize1
            local P = C.WorldPosition
            local Y = C.WorldPosition + C.WorldAxis * H
            local f = o.WorldPosition -o.WorldAxis * z
            local l = o.WorldPosition
            local Q, v, Z, p, F, a, t = clockFunc() -R, J.PulseLength, J.PulseSpeed, J.FadeLength, J.Color, J.ColorOffsetSpeed, 1 -J.ContractFrom
            if Q < (v + 1) / Z then
                C, b = P, P
                for g = 1, h, 1 do
                    local _ = L[g]
                    local L = g / h
                    H = calcPulseAlpha(L, Z, v, p, Q, N, A)
                    R, z = evalCubicBezier(P, Y, f, l, L), -Q
                    local N = s * z + d * 10 * L -0.2 + B * 4
                    local A = 5 * (s * 0.01 * z / 10 + d * L) + B * 4
                    local s, d, z = simplexNoise(5 * N, 1.5, 1 * A, 0, 0.6283185307179586) + simplexNoise(0.5 * N, 1.5, 0.1 * A, 0, 5.654866776461628), simplexNoise(3.4, A, N, q, W) * math.exp(- 5000 * (L -0.5) ^ 10), simplexNoise(2.3, A, N, w, u)
                    local q = g ~= h and(CFrame.new(C, R) * CFrame.Angles(0, 0, s) * CFrame.Angles(math.acos(math.clamp(simplexNoise(A, N, 2.7, 6.123233995736766E-17, 1), -1, 1)), 0, 0) * CFrame.new(0, 0, -d)).Position or R
                    if H > t then
                        _.Size = Vector3.new((q -b).Magnitude, U * z * H, U * z * H)
                        _.CFrame = CFrame.lookAt(0.5 * (b + q), q) * beamRotationOffset
                        _.Transparency = 1 -H
                    elseif H > t -1 / (h * p) then
                        o = (1 -(H -(t -1 / (h * p))) * h * p) * (L < Q * Z -0.5 * v and 1 or -1)
                        _.Size = Vector3.new((1 -math.abs(o)) * (q -b).Magnitude, U * z * H, U * z * H)
                        _.CFrame = CFrame.lookAt(b + (q -b) * (math.max(0, o) + 0.5 * (1 -math.abs(o))), q) * beamRotationOffset
                        _.Transparency = 1 -H
                    else
                        _.Transparency = 1
                    end
                    if typeof(F) == "Color3" then
                        _.Color = F
                    else
                        L = (B + L -Q * a) % 1
                        g = F.Keypoints
                        for M = 1, #g -1, 1 do
                            if g[M].Time < L and L < g[M + 1].Time then
                                _.Color = g[M].Value:Lerp(g[M + 1].Value, (L -g[M].Time) / (g[M + 1].Time -g[M].Time))
                                break
                            end
                        end
                    end
                    C, b = R, q
                end
            else
                J:Destroy()
            end
        elseif J.PartsHidden == false then
            J.PartsHidden = true
            local M = J.DisabledTransparency
            for b = 1, #J.Parts, 1 do
                J.Parts[b].Transparency = M
            end
        end
    end
end
end

-- ----------------------------------------------------------------------------
-- Function [89]: delayedCall10ms (79 bytes)
-- Description: Waits 0.01s via task.wait and invokes wrapped 2-arg function.
-- Resolved Upvalues: [1] = wrappedFunc
-- ----------------------------------------------------------------------------
do
    local wrappedFunc
    YabujinDeobfuscated.delayedCall10ms = function(b, J)
        task.wait(0.01)
        return wrappedFunc(b, J)
    end
end

-- ----------------------------------------------------------------------------
-- Function [98]: delayedCall20ms (73 bytes)
-- Description: Waits 0.02s via task.wait and invokes wrapped function.
-- Resolved Upvalues: [1] = wrappedFunc
-- ----------------------------------------------------------------------------
do
    local wrappedFunc
    YabujinDeobfuscated.delayedCall20ms = function(b)
        task.wait(0.02)
        return wrappedFunc(b)
    end
end

-- ----------------------------------------------------------------------------
-- Function [79]: delayedCall30ms (73 bytes)
-- Description: Waits 0.03s via task.wait and invokes wrapped function.
-- Resolved Upvalues: [1] = wrappedFunc
-- ----------------------------------------------------------------------------
do
    local wrappedFunc
    YabujinDeobfuscated.delayedCall30ms = function(b)
        task.wait(0.03)
        return wrappedFunc(b)
    end
end

-- ----------------------------------------------------------------------------
-- Function [83]: forwardOptionalCallback (92 bytes)
-- Description: Calls the current callback upvalue if assigned.
-- Resolved Upvalues: [1] = callbackRef
-- ----------------------------------------------------------------------------
do
    local callbackRef
    YabujinDeobfuscated.forwardOptionalCallback = function(...)
        local b = callbackRef
        if b then
            b(...)
        end
    end
end

-- ----------------------------------------------------------------------------
-- Function [122]: incrementActiveTickCounter (104 bytes)
-- Description: Increments tick counter upvalue when active flag is true.
-- Resolved Upvalues: [1] = isActive, [2] = tickCounter
-- ----------------------------------------------------------------------------
do
    local isActive, tickCounter
    YabujinDeobfuscated.incrementActiveTickCounter = function()
        if not isActive then return
    end
    tickCounter += 1
end
end

-- ============================================================================
-- PART 2: Embedded Obsidian ThemeManager ([71] — 11,147 bytes)
-- ============================================================================

YabujinDeobfuscated.initObsidianThemeManager = function()
        local b, J = cloneref or clonereference or function(N)
            return N
        end, clonefunction or copyfunction or function(N)
            return N
        end
        local N = b(game:GetService("HttpService"))
        local b = isfolder
        local A = isfile
        local q = listfiles
        if typeof(J) == "function" then
            local W, U, L = J(b), J(A), J(q)
            local J, h = pcall(function()
                return W("test" .. tostring(math.random(1000000, 9999999)))
            end)
            if J == false or typeof(h) ~= "boolean" then
                b = function(J)
                    local h, B = pcall(W, J)
                    return h and B or false
                end
                A = function(J)
                    local W, h = pcall(U, J)
                    return W and h or false
                end
                q = function(J)
                    local W, U = pcall(L, J)
                    return W and U or{}
                end
            end
        end
        local J = {}
        do
            local W = {"FontColor", "MainColor", "AccentColor", "BackgroundColor", "OutlineColor"}
            J.Folder = "ObsidianLibSettings"
            J.Library = nil
            J.AppliedToTab = false
            J.BuiltInThemes = {Default = {1, {FontColor = "dcdce4", MainColor = "1c1c22", AccentColor = "6382ff", BackgroundColor = "121216", OutlineColor = "2a2a32"}}, BBot = {2, {FontColor = "ffffff", MainColor = "1e1e1e", AccentColor = "7e48a3", BackgroundColor = "232323", OutlineColor = "141414"}}, Fatality = {3, {FontColor = "ffffff", MainColor = "1e1842", AccentColor = "c50754", BackgroundColor = "191335", OutlineColor = "3c355d"}}, Jester = {4, {FontColor = "ffffff", MainColor = "242424", AccentColor = "db4467", BackgroundColor = "1c1c1c", OutlineColor = "373737"}}, Mint = {5, {FontColor = "ffffff", MainColor = "242424", AccentColor = "3db488", BackgroundColor = "1c1c1c", OutlineColor = "373737"}}, ["Tokyo Night"] = {6, {FontColor = "ffffff", MainColor = "191925", AccentColor = "6759b3", BackgroundColor = "16161f", OutlineColor = "323232"}}, Ubuntu = {7, {FontColor = "ffffff", MainColor = "3e3e3e", AccentColor = "e2581e", BackgroundColor = "323232", OutlineColor = "191919"}}, Quartz = {8, {FontColor = "ffffff", MainColor = "232330", AccentColor = "426e87", BackgroundColor = "1d1b26", OutlineColor = "27232f"}}, Nord = {9, {FontColor = "eceff4", MainColor = "3b4252", AccentColor = "88c0d0", BackgroundColor = "2e3440", OutlineColor = "4c566a"}}, Dracula = {10, {FontColor = "f8f8f2", MainColor = "44475a", AccentColor = "ff79c6", BackgroundColor = "282a36", OutlineColor = "6272a4"}}, Monokai = {11, {FontColor = "f8f8f2", MainColor = "272822", AccentColor = "f92672", BackgroundColor = "1e1f1c", OutlineColor = "49483e"}}, Gruvbox = {12, {FontColor = "ebdbb2", MainColor = "3c3836", AccentColor = "fb4934", BackgroundColor = "282828", OutlineColor = "504945"}}, Solarized = {13, {FontColor = "839496", MainColor = "073642", AccentColor = "cb4b16", BackgroundColor = "002b36", OutlineColor = "586e75"}}, Catppuccin = {14, {FontColor = "d9e0ee", MainColor = "302d41", AccentColor = "f5c2e7", BackgroundColor = "1e1e2e", OutlineColor = "575268"}}, ["One Dark"] = {15, {FontColor = "abb2bf", MainColor = "282c34", AccentColor = "c678dd", BackgroundColor = "21252b", OutlineColor = "5c6370"}}, Cyberpunk = {16, {FontColor = "f9f9f9", MainColor = "262335", AccentColor = "00ff9f", BackgroundColor = "1a1a2e", OutlineColor = "413c5e"}}, ["Oceanic Next"] = {17, {FontColor = "d8dee9", MainColor = "1b2b34", AccentColor = "6699cc", BackgroundColor = "16232a", OutlineColor = "343d46"}}, Material = {18, {FontColor = "eeffff", MainColor = "212121", AccentColor = "82aaff", BackgroundColor = "151515", OutlineColor = "424242"}}, Minimal = {19, {FontColor = "dcdce4", MainColor = "1c1c22", AccentColor = "6382ff", BackgroundColor = "121216", OutlineColor = "2a2a32"}}}
            J.SetLibrary = function(U, L)
                U.Library = L
            end
            J.GetPaths = function(U)
                local L, h = {}, U.Folder:split("/")
                for B = 1, #h, 1 do
                    L[#L + 1] = table.concat(h, "/", 1, B)
                end
                L[#L + 1] = U.Folder .. "/themes"
                return L
            end
            J.BuildFolderTree = function(U)
                local L = U:GetPaths()
                for h = 1, #L, 1 do
                    U = L[h]
                    if not b(U) then
                        makefolder(U)
                    end
                end
            end
            J.CheckFolderTree = function(U)
                if b(U.Folder) then return
            end
            U:BuildFolderTree()
            task.wait(0.1)
        end
        J.SetFolder = function(b, U)
            b.Folder = U
            b:BuildFolderTree()
        end
        J.ApplyTheme = function(b, U)
            local L = b:GetCustomTheme(U)
            local h = L or b.BuiltInThemes[U]
            if not h then
                return
            end
            for U, B in pairs(L or h[2]) do
                if U ~= "VideoLink" then
                    if U == "FontFace" then
                        b.Library:SetFont(Enum.Font[B])
                        if b.Library.Options[U] then
                            b.Library.Options[U]:SetValue(B)
                        end
                    else
                        b.Library.Scheme[U] = Color3.fromHex(B)
                        if b.Library.Options[U] then
                            b.Library.Options[U]:SetValueRGB(Color3.fromHex(B))
                        end
                    end
                end
            end
            b:ThemeUpdate()
        end
        J.ThemeUpdate = function(b)
            for U, U in W, nil, nil do
                if b.Library.Options and b.Library.Options[U] then
                    b.Library.Scheme[U] = b.Library.Options[U].Value
                end
            end
            b.Library:UpdateColorsUsingRegistry()
        end
        J.GetCustomTheme = function(b, U)
            local L = b.Folder .. "/themes/" .. U .. ".json"
            if not A(L) then
                return nil
            end
            U = readfile(L)
            L, b = pcall(N.JSONDecode, N, U)
            if not L then
                return nil
            end
            return b
        end
        J.LoadDefault = function(b)
            local U, L, h = "Default", A(b.Folder .. "/themes/default.txt") and(readfile(b.Folder .. "/themes/default.txt")), true
            if L then
                if b.BuiltInThemes[L] then
                    U = L
                elseif b:GetCustomTheme(L) then
                    h, U = false, L
                end
            else
                U = if b.BuiltInThemes[b.DefaultTheme] then b.DefaultTheme else U
            end
            if h then
                b.Library.Options.ThemeManager_ThemeList:SetValue(U)
            else
                b:ApplyTheme(U)
            end
        end
        J.SaveDefault = function(b, U)
            writefile(b.Folder .. "/themes/default.txt", U)
        end
        J.SetDefaultTheme = function(b, U)
            assert(b.Library, "Must set ThemeManager.Library first!")
            assert(not b.AppliedToTab, "Cannot set default theme after applying ThemeManager to a tab!")
            local L, h = {}, {}
            for B, B in W, nil, nil do
                if typeof(U[B]) == "Color3" then
                    L[B] = "#" .. U[B]:ToHex()
                    h[B] = U[B]
                elseif typeof(U[B]) == "string" then
                    L[B] = U[B]:sub(1, 1) == "#" and U[B] or "#" .. U[B]
                    h[B] = Color3.fromHex(U[B])
                else
                    L[B] = J.BuiltInThemes.Default[2][B]
                    h[B] = Color3.fromHex(J.BuiltInThemes.Default[2][B])
                end
            end
            if typeof(U.FontFace) == "EnumItem" then
                L.FontFace = U.FontFace.Name
                h.Font = Font.fromEnum(U.FontFace)
            elseif typeof(U.FontFace) == "string" then
                L.FontFace = U.FontFace
                h.Font = Font.fromEnum(Enum.Font[U.FontFace])
            else
                L.FontFace = "Code"
                h.Font = Font.fromEnum(Enum.Font.Code)
            end
            U = {"RedColor", "DarkColor", "WhiteColor"}
            for B, B in U, nil, nil do
                h[B] = b.Library.Scheme[B]
            end
            b.Library.Scheme = h
            b.BuiltInThemes.Default = {1, L}
            b.Library:UpdateColorsUsingRegistry()
        end
        J.SaveCustomTheme = function(b, U)
            if U:gsub(" ", "") == "" then b.Library:Notify("Invalid file name for theme (empty)", 3)
            return
        end
        local L = {}
        for h, h in W, nil, nil do
            L[h] = b.Library.Options[h].Value:ToHex()
        end
        L.FontFace = b.Library.Options.FontFace.Value
        writefile(b.Folder .. "/themes/" .. U .. ".json", N:JSONEncode(L))
    end
    J.Delete = function(b, N)
        if not N then return false, "no config file is selected"
    end
    local W = b.Folder .. "/themes/" .. N .. ".json"
    if not A(W) then
        return false, "invalid file"
    end
    if not pcall(delfile, W) then
        return false, "delete file error"
    end
    return true
end
J.ReloadCustomThemes = function(b)
    local N, A = q(b.Folder .. "/themes"), {}
    for q = 1, #N, 1 do
        b = N[q]
        if b:sub(- 5) == ".json" then
            q = b:find(".json", 1, true)
            local N, W = (b:sub(q, q)), q
            while N ~= "/" and N ~= "\\" and N ~= "" do
                q -= 1
                N = (b:sub(q, q))
            end
            if N == "/" or N == "\\" then
                table.insert(A, b:sub(q + 1, W -1))
            end
        end
    end
    return A
end
J.CreateThemeManager = function(b, N)
    N:AddLabel("Background color"):AddColorPicker("BackgroundColor", {Default = b.Library.Scheme.BackgroundColor})
    N:AddLabel("Main color"):AddColorPicker("MainColor", {Default = b.Library.Scheme.MainColor})
    N:AddLabel("Accent color"):AddColorPicker("AccentColor", {Default = b.Library.Scheme.AccentColor})
    N:AddLabel("Outline color"):AddColorPicker("OutlineColor", {Default = b.Library.Scheme.OutlineColor})
    N:AddLabel("Font color"):AddColorPicker("FontColor", {Default = b.Library.Scheme.FontColor})
    N:AddDropdown("FontFace", {Text = "Font Face", Default = "Code", Values = {"BuilderSans", "Code", "Fantasy", "Gotham", "Jura", "Roboto", "RobotoMono", "SourceSans"}})
    local A = {}
    for q, W in pairs(b.BuiltInThemes) do
        table.insert(A, q)
    end
    table.sort(A, function(q, W)
        return b.BuiltInThemes[q][1] < b.BuiltInThemes[W][1]
    end)
    N:AddDivider()
    N:AddDropdown("ThemeManager_ThemeList", {Text = "Theme list", Values = A, Default = 1})
    N:AddButton("Set as default", function()
        b:SaveDefault(b.Library.Options.ThemeManager_ThemeList.Value)
        b.Library:Notify(string.format("Set default theme to %q", b.Library.Options.ThemeManager_ThemeList.Value))
    end)
    b.Library.Options.ThemeManager_ThemeList:OnChanged(function()
        b:ApplyTheme(b.Library.Options.ThemeManager_ThemeList.Value)
    end)
    N:AddDivider()
    N:AddInput("ThemeManager_CustomThemeName", {Text = "Custom theme name"})
    N:AddButton("Create theme", function()
        local A = b.Library.Options.ThemeManager_CustomThemeName.Value
        if A:gsub(" ", "") == "" then
            b.Library:Notify("Invalid theme name (empty)", 2)
            return
        end
        b:SaveCustomTheme(A)
        b.Library:Notify(string.format("Created theme %q", A))
        b.Library.Options.ThemeManager_CustomThemeList:SetValues(b:ReloadCustomThemes())
        b.Library.Options.ThemeManager_CustomThemeList:SetValue(nil)
    end)
    N:AddDivider()
    N:AddDropdown("ThemeManager_CustomThemeList", {Text = "Custom themes", Values = b:ReloadCustomThemes(), AllowNull = true, Default = 1})
    N:AddButton("Load theme", function()
        local A = b.Library.Options.ThemeManager_CustomThemeList.Value
        b:ApplyTheme(A)
        b.Library:Notify(string.format("Loaded theme %q", A))
    end)
    N:AddButton("Overwrite theme", function()
        local A = b.Library.Options.ThemeManager_CustomThemeList.Value
        b:SaveCustomTheme(A)
        b.Library:Notify(string.format("Overwrote config %q", A))
    end)
    N:AddButton("Delete theme", function()
        local A = b.Library.Options.ThemeManager_CustomThemeList.Value
        local q, W = b:Delete(A)
        if not q then
            b.Library:Notify("Failed to delete theme: " .. W)
            return
        end
        b.Library:Notify(string.format("Deleted theme %q", A))
        b.Library.Options.ThemeManager_CustomThemeList:SetValues(b:ReloadCustomThemes())
        b.Library.Options.ThemeManager_CustomThemeList:SetValue(nil)
    end)
    N:AddButton("Refresh list", function()
        b.Library.Options.ThemeManager_CustomThemeList:SetValues(b:ReloadCustomThemes())
        b.Library.Options.ThemeManager_CustomThemeList:SetValue(nil)
    end)
    N:AddButton("Set as default", function()
        if b.Library.Options.ThemeManager_CustomThemeList.Value ~= nil and b.Library.Options.ThemeManager_CustomThemeList.Value ~= "" then b:SaveDefault(b.Library.Options.ThemeManager_CustomThemeList.Value)
        b.Library:Notify(string.format("Set default theme to %q", b.Library.Options.ThemeManager_CustomThemeList.Value))
    end
end)
N:AddButton("Reset default", function()
    if not pcall(delfile, b.Folder .. "/themes/default.txt") then b.Library:Notify("Failed to reset default: delete file error")
    return
end
b.Library:Notify("Set default theme to nothing")
b.Library.Options.ThemeManager_CustomThemeList:SetValues(b:ReloadCustomThemes())
b.Library.Options.ThemeManager_CustomThemeList:SetValue(nil)
end)
b:LoadDefault()
b.AppliedToTab = true
local function M()
    b:ThemeUpdate()
end
b.Library.Options.BackgroundColor:OnChanged(M)
b.Library.Options.MainColor:OnChanged(M)
b.Library.Options.AccentColor:OnChanged(M)
b.Library.Options.OutlineColor:OnChanged(M)
b.Library.Options.FontColor:OnChanged(M)
b.Library.Options.FontFace:OnChanged(function(M)
    b.Library:SetFont(Enum.Font[M])
    b.Library:UpdateColorsUsingRegistry()
end)
end
J.CreateGroupBox = function(M, b)
    assert(M.Library, "Must set ThemeManager.Library first!")
    return b:AddLeftGroupbox("Themes", "paintbrush")
end
J.ApplyToTab = function(M, b)
    assert(M.Library, "Must set ThemeManager.Library first!")
    M:CreateThemeManager((M:CreateGroupBox(b)))
end
J.ApplyToGroupbox = function(M, b)
    assert(M.Library, "Must set ThemeManager.Library first!")
    M:CreateThemeManager(b)
end
J:BuildFolderTree()
end
getgenv().ObsidianThemeManager = J
return J
end

-- ============================================================================
-- PART 3: Embedded YabujinLibrary / Obsidian UI Library ([56] — 146,597 bytes)
-- ============================================================================

do
    local yabujinCoreState = {}
    YabujinDeobfuscated.initYabujinObsidianLibrary = function()
        local b = Instance.new
        yabujinCoreState._instNew = b
        local J = cloneref or clonereference or function(N)
            return N
        end
        local N, A, q, W, U, L, h, B, R, s, d = J(game:GetService("CoreGui")), J(game:GetService("Players")), J(game:GetService("RunService")), J(game:GetService("SoundService")), J(game:GetService("UserInputService")), J(game:GetService("TextService")), J(game:GetService("Teams")), J(game:GetService("TweenService")), getgenv or function()
            return shared
        end, setclipboard or nil, protectgui or syn and syn.protect_gui or function()
        end
        local w, u = gethui or function()
            return N
        end, A.LocalPlayer or(A.PlayerAdded:Wait())
        local C = J(u:GetMouse())
        local J = {}
        local o = {}
        local H = {}
        local z = {}
        local P = {}
        local Y = {}
        local f = {TransparencyTexture = {RobloxId = 139785960036434, Path = "Obsidian/assets/TransparencyTexture.png", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/TransparencyTexture.png", Id = nil}, SaturationMap = {RobloxId = 4155801252, Path = "Obsidian/assets/SaturationMap.png", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/SaturationMap.png", Id = nil}, ShadowImage = {RobloxId = 6015897843, Path = "Obsidian/assets/shadow.png", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/shadow.png", Id = nil}, HoverSound = {RobloxId = 6895079853, Path = "Obsidian/assets/sounds/hover.wav", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/sounds/hover.wav", Id = nil}, ClickSound = {RobloxId = 6895079853, Path = "Obsidian/assets/sounds/click.wav", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/sounds/click.wav", Id = nil}, ToggleOnSound = {RobloxId = 6895079853, Path = "Obsidian/assets/sounds/toggle_on.wav", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/sounds/toggle_on.wav", Id = nil}, ToggleOffSound = {RobloxId = 6895079853, Path = "Obsidian/assets/sounds/toggle_off.wav", URL = "https://raw.githubusercontent.com/deividcomsono/Obsidian/refs/heads/main/assets/sounds/toggle_off.wav", Id = nil}}
        do
            local function l(Q, v)
                if not isfolder or not makefolder then return
            end
            local Z = Q:split("/")
            if v then
                table.remove(Z, #Z)
            end
            Q = ""
            for v, v in ipairs(Z) do
                if not isfolder(Q .. v) then
                    makefolder(Q .. v)
                end
                Q ..= v .. "/"
            end
            return Q
        end
        Y.AddAsset = function(Q, v, Z, p)
            if f[Q] ~= nil then error(string.format("Asset %q already exists", Q))
        end
        assert(typeof(v) == "number", "RobloxAssetId must be a number")
        f[Q] = {RobloxId = v, Path = string.format("Obsidian/custom_assets/%s", Q), URL = Z, Id = nil}
        Y.DownloadAsset(Q, p)
    end
    Y.GetAsset = function(Q)
        if not f[Q] then return nil
    end
    local v = f[Q]
    if v.Id then
        return v.Id
    end
    Q = string.format("rbxassetid://%s", v.RobloxId)
    if getcustomasset then
        local Z, p = pcall(getcustomasset, v.Path)
        Q = if Z and p then p else Q
    end
    v.Id = Q
    return Q
end
Y.DownloadAsset = function(Q, v)
    if not getcustomasset or not writefile or not isfile then return false, "missing functions"
end
local Z = f[Q]
l(Z.Path, true)
if v ~= true and(isfile(Z.Path)) then
    return true, nil
end
Q, v = pcall(function()
    local l
    if(identifyexecutor and(identifyexecutor()) or "Unknown") == "Wave" and request then
        local p, F = pcall(request, {Url = Z.URL, Method = "GET"})
        l = if p and F and F.Body then F.Body else l
    end
    writefile(Z.Path, if not l then(game:HttpGet(Z.URL)) else l)
end)
return Q, v
end
for l, Q in pairs(f) do
    Y.DownloadAsset(l)
end
end
local f = {LocalPlayer = u, DevicePlatform = nil, IsMobile = false, IsRobloxFocused = true, ScreenGui = nil, SearchText = "", Searching = false, GlobalSearch = false, LastSearchTab = nil, ActiveTab = nil, Tabs = {}, TabButtons = {}, DependencyBoxes = {}, KeybindFrame = nil, KeybindContainer = nil, KeybindToggles = {}, Notifications = {}, ToggleKeybind = Enum.KeyCode.RightControl, TweenInfo = TweenInfo.new(0.18, Enum.EasingStyle.Exponential, Enum.EasingDirection.Out), NotifyTweenInfo = TweenInfo.new(0.35, Enum.EasingStyle.Exponential, Enum.EasingDirection.Out), Toggled = false, Unloaded = false, Labels = J, Buttons = o, Toggles = H, Options = z, NotifySide = "Right", ShowCustomCursor = true, ForceCheckbox = false, ShowToggleFrameInKeybinds = true, NotifyOnError = false, CantDragForced = false, Signals = {}, UnloadSignals = {}, OriginalMinSize = Vector2.new(480, 360), MinSize = Vector2.new(480, 360), DPIScale = 1, CornerRadius = 12, IsLightTheme = false, Scheme = {BackgroundColor = Color3.fromRGB(13, 14, 18), MainColor = Color3.fromRGB(21, 22, 28), AccentColor = Color3.fromRGB(150, 120, 255), OutlineColor = Color3.fromRGB(34, 36, 46), FontColor = Color3.fromRGB(236, 237, 244), Font = (function()
    local l, Q = pcall(function()
        return Font.fromEnum(Enum.Font.BuilderSansMedium)
    end)
    if l and Q then
        return Q
    end
    return Font.fromEnum(Enum.Font.GothamMedium)
end)(), RedColor = Color3.fromRGB(255, 70, 90), DarkColor = Color3.new(0, 0, 0), WhiteColor = Color3.new(1, 1, 1)}, Registry = {}, Scales = {}, ImageManager = Y}
if q:IsStudio() then
    if U.TouchEnabled and not U.MouseEnabled then
        f.IsMobile = true
        f.OriginalMinSize = Vector2.new(480, 240)
    else
        f.IsMobile = false
        f.OriginalMinSize = Vector2.new(480, 360)
    end
else
    pcall(function()
        f.DevicePlatform = U:GetPlatform()
    end)
    f.IsMobile = f.DevicePlatform == Enum.Platform.Android or f.DevicePlatform == Enum.Platform.IOS
    f.OriginalMinSize = f.IsMobile and(Vector2.new(480, 240)) or(Vector2.new(480, 360))
end
local l, Q, v, Z, p = {Frame = {BorderSizePixel = 0}, ImageLabel = {BackgroundTransparency = 1, BorderSizePixel = 0}, ImageButton = {AutoButtonColor = false, BorderSizePixel = 0}, ScrollingFrame = {BorderSizePixel = 0}, TextLabel = {BorderSizePixel = 0, FontFace = "Font", RichText = true, TextColor3 = "FontColor"}, TextButton = {AutoButtonColor = false, BorderSizePixel = 0, FontFace = "Font", RichText = true, TextColor3 = "FontColor"}, TextBox = {BorderSizePixel = 0, FontFace = "Font", PlaceholderColor3 = function()
    local F, a, t = f.Scheme.FontColor:ToHSV()
    return Color3.fromHSV(F, a, t / 2)
end, Text = "", TextColor3 = "FontColor"}, UIListLayout = {SortOrder = Enum.SortOrder.LayoutOrder}, UIStroke = {ApplyStrokeMode = Enum.ApplyStrokeMode.Border}, Window = {Title = "No Title", Footer = "No Footer", Position = UDim2.fromOffset(6, 6), Size = UDim2.fromOffset(720, 600), IconSize = UDim2.fromOffset(30, 30), AutoShow = true, Center = true, Resizable = true, SearchbarSize = UDim2.fromScale(1, 1), GlobalSearch = false, CornerRadius = 8, NotifySide = "Right", ShowCustomCursor = true, Font = Enum.Font.GothamMedium, ToggleKeybind = Enum.KeyCode.RightControl, MobileButtonsSide = "Left", UnlockMouseWhileOpen = true, EnableBlur = true, EnableSnowfall = false, GlassType = "None", EnableFadeIn = true, EnableSidebarResize = false, EnableCompacting = true, DisableCompactingSnap = false, SidebarCompacted = false, MinContainerWidth = 256, MinSidebarWidth = 128, SidebarCompactWidth = 48, SidebarCollapseThreshold = 0.5, CompactWidthActivation = 128}, Toggle = {Text = "Toggle", Default = false, Callback = function()
end, Changed = function()
end, Risky = false, Disabled = false, Visible = true}, Input = {Text = "Input", Default = "", Finished = false, Numeric = false, ClearTextOnFocus = true, Placeholder = "", AllowEmpty = true, EmptyReset = "---", Callback = function()
end, Changed = function()
end, Disabled = false, Visible = true}, Slider = {Text = "Slider", Default = 0, Min = 0, Max = 100, Rounding = 0, Prefix = "", Suffix = "", Callback = function()
end, Changed = function()
end, Disabled = false, Visible = true}, Dropdown = {Values = {}, DisabledValues = {}, Multi = false, MaxVisibleDropdownItems = 8, Callback = function()
end, Changed = function()
end, Disabled = false, Visible = true}, Viewport = {Object = nil, Camera = nil, Clone = true, AutoFocus = true, Interactive = false, Zoom = true, Height = 200, Visible = true}, Image = {Image = "", Transparency = 0, BackgroundTransparency = 0, Color = Color3.new(1, 1, 1), RectOffset = Vector2.zero, RectSize = Vector2.zero, ScaleType = Enum.ScaleType.Fit, Height = 200, Visible = true}, Video = {Video = "", Looped = false, Playing = false, Volume = 1, Height = 200, Visible = true}, UIPassthrough = {Instance = nil, Height = 24, Visible = true}, KeyPicker = {Text = "KeyPicker", Default = "None", DefaultModifiers = {}, Mode = "Toggle", Modes = {"Always", "Toggle", "Hold"}, SyncToggleState = false, Callback = function()
end, ChangedCallback = function()
end, Changed = function()
end, Clicked = function()
end
}, ColorPicker = {Default = Color3.new(1, 1, 1), Callback = function()
end, Changed = function()
end
}}, {Bottom = {0, 1}, Right = {1, 0}}, {Left = {0.5, 1}, Right = {0.5, 1}}, {RedColor = "Red", WhiteColor = "White", DarkColor = "Dark"}, {Red = "RedColor", White = "WhiteColor", Dark = "DarkColor"}
local function F(a)
    if not a then return nil
end
local t = Z[a]
if t and f.Scheme[t] ~= nil then
    f.Scheme[a] = f.Scheme[t]
    f.Scheme[t] = nil
    return f.Scheme[a]
end
t = p[a]
if t and f.Scheme[t] ~= nil then
    warn(string.format("Scheme Value %q is deprecated, please use %q instead.", a, t))
    return f.Scheme[t]
end
return f.Scheme[a]
end
local function Z(p, a, t)
    local g = Instance.new("BindableEvent")
    local _ = p:Once(function(...)
        if not t or typeof(t) == "function" and(t(...)) then g:Fire(true)
    else
        g:Fire(false)
    end
end)
task.delay(a, function()
    _:Disconnect()
    g:Fire(false)
end)
p = g.Event:Wait()
g:Destroy()
return p
end
local function p(a, t)
    return a.UserInputType == Enum.UserInputType.MouseButton1 or t == true and a.UserInputType == Enum.UserInputType.MouseButton2 or a.UserInputType == Enum.UserInputType.Touch
end
local function a(t, g)
    return p(t, g) and t.UserInputState == Enum.UserInputState.Begin and f.IsRobloxFocused
end
local function t(g)
    return(g.UserInputType == Enum.UserInputType.MouseMovement or g.UserInputType == Enum.UserInputType.Touch) and g.UserInputState == Enum.UserInputState.Change
end
local function g(_, c)
    return p(_, c) and(_.UserInputState == Enum.UserInputState.Begin or _.UserInputState == Enum.UserInputState.Change) and f.IsRobloxFocused
end
local function p(_)
    local c = 0
    for e, e in pairs(_) do
        c += 1
    end
    return c
end
local function _(c)
    if not(c and c.PlaybackState == Enum.PlaybackState.Playing) then return
end
c:Cancel()
end
local function c(e)
    return e:match("^%s*(.-)%s*$")
end
local function e(r, I)
    assert(I >= 0, "Invalid rounding number.")
    if I == 0 then
        return math.floor(r)
    end
    return tonumber(string.format("%." .. I .. "f", r))
end
local function r(I)
    local x = A:GetPlayers()
    if I then
        local I = table.find(x, u)
        if I then
            table.remove(x, I)
        end
    end
    table.sort(x, function(u, I)
        return u.Name:lower() < I.Name:lower()
    end)
    return x
end
local function u()
    local I = h:GetTeams()
    table.sort(I, function(x, D)
        return x.Name:lower() < D.Name:lower()
    end)
    return I
end
f.UpdateDependencyBoxes = function(I)
    for I, I in pairs(f.DependencyBoxes) do
        I:Update(true)
    end
    if f.Searching then
        f:UpdateSearch(f.SearchText)
    end
end
local function I(x, D)
    local X = 0
    for V, K in pairs(x.Elements) do
        if K.Type == "Divider" then
            K.Holder.Visible = false
        elseif K.SubButton then
            local j, O = false, K.Text:lower():match(D) and K.Visible
            if O then
                j = true
            else
                K.Base.Visible = false
            end
            V = K.SubButton.Text:lower():match(D) and K.SubButton.Visible
            if V then
                j = true
            else
                K.SubButton.Base.Visible = false
            end
            K.Holder.Visible = j
            X = if j then X + 1 else X
        elseif K.Text and(K.Text:lower():match(D)) and K.Visible then
            K.Holder.Visible = true
            X += 1
        else
            K.Holder.Visible = false
        end
    end
    for V, V in pairs(x.DependencyBoxes) do
        X = if V.Visible then X + I(V, D) else X
    end
    x.Holder.Visible = X > 0
    return X
end
local function x(D)
    for X, X in pairs(D.Elements) do
        X.Holder.Visible = typeof(X.Visible) == "boolean" and X.Visible or true
        if X.SubButton then
            X.Base.Visible = X.Visible
            X.SubButton.Base.Visible = X.SubButton.Visible
        end
    end
    D:Resize()
    D.Holder.Visible = true
    for X, X in pairs(D.DependencyBoxes) do
        if X.Visible then
            x(X)
        end
    end
end
local function D(X, V)
    if not X then return
end
local K = false
for j, O in pairs(X.Groupboxes) do
    j = 0
    for m, k in pairs(O.Elements) do
        if k.Type == "Divider" then
            k.Holder.Visible = false
        elseif k.SubButton then
            local n, E = false, k.Text:lower():match(V) and k.Visible
            if E then
                n = true
            else
                k.Base.Visible = false
            end
            m = k.SubButton.Text:lower():match(V) and k.SubButton.Visible
            if m then
                n = true
            else
                k.SubButton.Base.Visible = false
            end
            k.Holder.Visible = n
            j = if n then j + 1 else j
        elseif k.Text and(k.Text:lower():match(V)) and k.Visible then
            k.Holder.Visible = true
            j += 1
        else
            k.Holder.Visible = false
        end
    end
    for m, m in pairs(O.DependencyBoxes) do
        j = if m.Visible then j + I(m, V) else j
    end
    if j > 0 then
        O:Resize()
        K = true
    end
    O.BoxHolder.Visible = j > 0
end
for j, O in pairs(X.Tabboxes) do
    j = {}
    for X, m in pairs(O.Tabs) do
        j[m] = 0
        for k, n in pairs(m.Elements) do
            if n.Type == "Divider" then
                n.Holder.Visible = false
            elseif n.SubButton then
                X, k = false, n.Text:lower():match(V) and n.Visible
                if k then
                    X = true
                else
                    n.Base.Visible = false
                end
                local k = n.SubButton.Text:lower():match(V) and n.SubButton.Visible
                if k then
                    X = true
                else
                    n.SubButton.Base.Visible = false
                end
                n.Holder.Visible = X
                if X then
                    j[m] = j[m] + 1
                end
            elseif n.Text and(n.Text:lower():match(V)) and n.Visible then
                n.Holder.Visible = true
                j[m] = j[m] + 1
            else
                n.Holder.Visible = false
            end
        end
        for X, X in pairs(m.DependencyBoxes) do
            if X.Visible then
                j[m] = j[m] + I(X, V)
            end
        end
    end
    local I = 0
    for X, V in pairs(j) do
        X.ButtonHolder.Visible = V > 0
        if V > 0 then
            I += 1
            if O.ActiveTab == X then
                X:Resize()
            elseif O.ActiveTab and j[O.ActiveTab] == 0 then
                X:Show()
            end
            K = true
        end
    end
    O.BoxHolder.Visible = I > 0
end
return K
end
local function I(X)
    if not X then return
end
for V, V in pairs(X.Groupboxes) do
    for K, K in pairs(V.Elements) do
        K.Holder.Visible = typeof(K.Visible) == "boolean" and K.Visible or true
        if K.SubButton then
            K.Base.Visible = K.Visible
            K.SubButton.Base.Visible = K.SubButton.Visible
        end
    end
    for K, K in pairs(V.DependencyBoxes) do
        if K.Visible then
            x(K)
        end
    end
    V:Resize()
    V.BoxHolder.Visible = true
end
for V, V in pairs(X.Tabboxes) do
    for X, X in pairs(V.Tabs) do
        for K, K in pairs(X.Elements) do
            K.Holder.Visible = typeof(K.Visible) == "boolean" and K.Visible or true
            if K.SubButton then
                K.Base.Visible = K.Visible
                K.SubButton.Base.Visible = K.SubButton.Visible
            end
        end
        for K, K in pairs(X.DependencyBoxes) do
            if K.Visible then
                x(K)
            end
        end
        X.ButtonHolder.Visible = true
    end
    if V.ActiveTab then
        V.ActiveTab:Resize()
    end
    V.BoxHolder.Visible = true
end
end
f.UpdateSearch = function(x, X)
    f.SearchText = X
    x = {}
    if f.GlobalSearch then
        for V, V in pairs(f.Tabs) do
            if typeof(V) == "table" and not V.IsKeyTab then
                table.insert(x, V)
            end
        end
    elseif f.LastSearchTab and typeof(f.LastSearchTab) == "table" then
        table.insert(x, f.LastSearchTab)
    end
    for V, V in ipairs(x) do
        I(V)
    end
    local I = X:lower()
    if c(I) == "" then
        f.Searching = false
        f.LastSearchTab = nil
        return
    end
    if not f.GlobalSearch and f.ActiveTab and f.ActiveTab.IsKeyTab then
        f.Searching = false
        f.LastSearchTab = nil
        return
    end
    f.Searching = true
    local V = {}
    if f.GlobalSearch then
        if#x == 0 then
            for K, K in pairs(f.Tabs) do
                if typeof(K) == "table" and not K.IsKeyTab then
                    table.insert(x, K)
                end
            end
        end
        V = x
    elseif f.ActiveTab then
        table.insert(V, f.ActiveTab)
    end
    local K
    x = false
    for j, j in ipairs(V) do
        if D(j, I) then
            if not K then
                K = j
            end
            x = if j == f.ActiveTab then true else x
        end
    end
    if f.GlobalSearch then
        if x and f.ActiveTab then
            f.ActiveTab:RefreshSides()
        elseif K then
            local I = X
            task.defer(function()
                if f.SearchText ~= I then return
            end
            if f.ActiveTab ~= K then
                K:Show()
            end
        end)
    end
    f.LastSearchTab = nil
else
    f.LastSearchTab = f.ActiveTab
end
end
f.AddToRegistry = function(I, I, x)
    f.Registry[I] = x
end
f.RemoveFromRegistry = function(I, I)
    f.Registry[I] = nil
end
f.UpdateColorsUsingRegistry = function(I)
    for x, D in pairs(f.Registry) do
        for X, V in pairs(D) do
            I = F(V)
            if I or typeof(V) == "function" then
                x[X] = I or(V())
            end
        end
    end
end
f.SetDPIScale = function(I, I)
    f.DPIScale = I / 100
    f.MinSize = f.OriginalMinSize * f.DPIScale
    for I, I in pairs(f.Scales) do
        I.Scale = f.DPIScale
    end
    for I, I in pairs(z) do
        if I.Type == "Dropdown" then
            I:RecalculateListSize()
        end
    end
    for I, I in pairs(f.Notifications) do
        I:Resize()
    end
end
f.GiveSignal = function(I, x)
    I = typeof(x)
    if x and(I == "RBXScriptConnection" or I == "RBXScriptSignal") then
        table.insert(f.Signals, x)
    end
    return x
end
f.PlaySound = function(I, I, I, I)
end
f.LoadSounds = function(I)
end
function IsValidCustomIcon(I)
    return typeof(I) == "string" and(I:match("rbxasset") or(I:match("roblox%.com/asset/%?id=")) or(I:match("rbxthumb://type=")))
end
local I, x = false
do
    local D, X = upval_3 .. "/Lucide/source.lua"
    local function V(K)
        if X then return
    end
    if request then
        pcall(function()
            local j = request({Url = K, Method = "GET"})
            local O = j and j.Body and#j.Body > 100
            if O then
                X = j.Body
            end
        end)
    end
    if not X then
        pcall(function()
            local j = game:HttpGet(K)
            local K = j and#j > 100
            if K then
                X = j
            end
        end)
    end
end
V(D)
V("https://raw.githubusercontent.com/deividcomsono/lucide-roblox-direct/refs/heads/main/source.lua")
if X then
    pcall(function()
        X = X:gsub("https://raw%.githubusercontent%.com/deividcomsono/lucide%-roblox%-direct/refs/heads/main/spritesheets/", upval_3 .. "/Lucide/spritesheets/")
    end)
    pcall(function()
        local D = loadstring(X)
        if D then
            local X = D()
            if X and X.GetAsset then
                I, x = true, X
            end
        end
    end)
end
end
f.GetIcon = function(D, D)
    if not I or not x then return
end
local X, V = pcall(x.GetAsset, D)
if not X or not V then
    return
end
return V
end
f.GetCustomIcon = function(D, D)
    if not IsValidCustomIcon(D) then return f:GetIcon(D)
else
    return{Url = D, ImageRectOffset = Vector2.zero, ImageRectSize = Vector2.zero, Custom = true}
end
end
f.Validate = function(D, D, X)
    if typeof(D) ~= "table" then return X
end
for V, K in pairs(X) do
    if typeof(V) ~= "number" then
        if typeof(K) == "table" then
            D[V] = f:Validate(D[V], K)
        elseif D[V] == nil then
            D[V] = K
        end
    end
end
return D
end
local D = {ScreenGui = true, SurfaceGui = true, BillboardGui = true, Lighting = true, UIScale = true, UIListLayout = true, UIGridLayout = true, UITableLayout = true, UIPageLayout = true, UIAspectRatioConstraint = true, UISizeConstraint = true, UITextSizeConstraint = true, UICorner = true, UIPadding = true, UIGradient = true, UIStroke = true, UIFlexItem = true}
local function X(V, K)
    local j = f.Registry[K] or{}
    for O, m in pairs(V) do
        if j[O] then
            j[O] = nil
        elseif O ~= "Text" then
            local V = F(m)
            if V or typeof(m) == "function" then
                j[O] = m
                m = V or(m())
            end
        end
        K[O] = m
    end
    if p(j) > 0 then
        f.Registry[K] = j
    end
end
local function F(V, K)
    local j = b(V)
    if l[V] then
        X(l[V], j)
    end
    X(K, j)
    if K.Parent and not K.ZIndex and not D[V] and not D[K.Parent.ClassName] then
        pcall(function()
            j.ZIndex = K.Parent.ZIndex
        end)
    end
    return j
end
local function b(D, X)
    if not(pcall(function()
        local V = X
        if not V then
            X = N
        end
        D.Parent = if typeof(X) == "function" then(X()) else X
    end) and D.Parent) then
        D.Parent = f.LocalPlayer:WaitForChild("PlayerGui", math.huge)
    end
end
local function D(X, V)
    if V then b(X, N)
    return
end
pcall(d, X)
b(X, w)
end
local b = F("ScreenGui", {Name = "Obsidian", DisplayOrder = 999, ResetOnSpawn = false})
D(b)
f.ScreenGui = b
b.DescendantRemoving:Connect(function(N)
    f:RemoveFromRegistry(N)
end)
local N = F("TextButton", {BackgroundTransparency = 1, Modal = false, Size = UDim2.fromScale(0, 0), AnchorPoint = Vector2.zero, Text = "", ZIndex = -999, Parent = b})
local d, w
do
    d = F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "WhiteColor", Size = UDim2.fromOffset(9, 1), Visible = false, ZIndex = 999, Parent = b})
    F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "DarkColor", Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.new(1, 2, 1, 2), ZIndex = 998, Parent = d})
    D = F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "WhiteColor", Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.fromOffset(1, 9), Parent = d})
    F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "DarkColor", Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.new(1, 2, 1, 2), ZIndex = 998, Parent = D})
    w = F("ImageLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.fromOffset(20, 20), ZIndex = 1000, Visible = false, Parent = d})
end
local D
do
    D = F("Frame", {AnchorPoint = Vector2.new(1, 0), BackgroundTransparency = 1, Position = UDim2.new(1, -6, 0, 6), Size = UDim2.new(0, 300, 1, -6), Parent = b})
    table.insert(f.Scales, (F("UIScale", {Parent = D})))
    F("UIListLayout", {HorizontalAlignment = Enum.HorizontalAlignment.Right, Padding = UDim.new(0, 8), Parent = D})
end
f.ResetCursorIcon = function(X)
    w.Visible = false
    w.Size = UDim2.fromOffset(20, 20)
end
f.ChangeCursorIcon = function(X, V)
    if not V or V == "" then f:ResetCursorIcon()
    return
end
X = f:GetCustomIcon(V)
assert(X, "Image must be a valid Roblox asset or a valid URL or a valid lucide icon.")
w.Visible = true
w.Image = X.Url
w.ImageRectOffset = X.ImageRectOffset
w.ImageRectSize = X.ImageRectSize
end
f.ChangeCursorIconSize = function(X, X)
    assert(typeof(X) == "UDim2", "UDim2 expected.")
    w.Size = X
end
f.GetBetterColor = function(X, X, V)
    V*=f.IsLightTheme and -4 or 2
    return Color3.fromRGB(math.clamp(X.R * 255 + V, 0, 255), math.clamp(X.G * 255 + V, 0, 255), math.clamp(X.B * 255 + V, 0, 255))
end
f.GetLighterColor = function(X, X)
    local V, K, j = X:ToHSV()
    return Color3.fromHSV(V, math.max(0, K -0.1), math.min(1, j + 0.1))
end
f.GetDarkerColor = function(X, X)
    local V, K, j = X:ToHSV()
    return Color3.fromHSV(V, K, j / 2)
end
f.GetKeyString = function(X, X)
    if X.EnumType == Enum.KeyCode and X.Value > 33 and X.Value < 127 then return string.char(X.Value)
end
return X.Name
end
f.GetTextBounds = function(X, V, K, j, O)
    X = Instance.new("GetTextBoundsParams")
    X.Text = V
    X.RichText = true
    X.Font = K
    X.Size = j
    X.Width = O or workspace.CurrentCamera.ViewportSize.X -32
    j = L:GetTextBoundsAsync(X)
    return j.X, j.Y
end
f.MouseIsOverFrame = function(L, L, X)
    local V, K = L.AbsolutePosition, L.AbsoluteSize
    return X.X >= V.X and X.X <= V.X + K.X and X.Y >= V.Y and X.Y <= V.Y + K.Y
end
f.SafeCallback = function(L, X, ...)
    if not(X and typeof(X) == "function") then return
end
L = table.pack(xpcall(X, function(X)
    task.defer(error, debug.traceback(X, 2))
    if f.NotifyOnError then
        f:Notify(X)
    end
    return X
end, ...))
if not L[1] then
    return nil
end
return table.unpack(L, 2, L.n)
end
f.MakeDraggable = function(L, L, X, V, K)
    local j, O
    local m, k = false
    X.InputBegan:Connect(function(X)
        if not a(X) or K and f.CantDragForced then return
    end
    j = X.Position
    O, m = L.Position, true
    k = X.Changed:Connect(function()
        if X.UserInputState ~= Enum.UserInputState.End then return
    end
    m = false
    if k and k.Connected then
        k:Disconnect()
        k = nil
    end
end)
end)
f:GiveSignal(U.InputChanged:Connect(function(X)
    if not V and not f.Toggled or K and f.CantDragForced or not(b and b.Parent) then m = false
    if k and k.Connected then
        k:Disconnect()
        k = nil
    end
    return
end
if m and(t(X)) then
    local V = X.Position -j
    L.Position = UDim2.new(O.X.Scale, O.X.Offset + V.X, O.Y.Scale, O.Y.Offset + V.Y)
end
end))
end
f.MakeResizable = function(L, L, X, V)
    local K, j
    local O, m = false
    X.InputBegan:Connect(function(X)
        if not a(X) then return
    end
    K = X.Position
    j, O = L.Size, true
    m = X.Changed:Connect(function()
        if X.UserInputState ~= Enum.UserInputState.End then return
    end
    O = false
    if m and m.Connected then
        m:Disconnect()
        m = nil
    end
end)
end)
f:GiveSignal(U.InputChanged:Connect(function(X)
    if not L.Visible or not(b and b.Parent) then O = false
    if m and m.Connected then
        m:Disconnect()
        m = nil
    end
    return
end
if O and(t(X)) then
    local O = X.Position -K
    L.Size = UDim2.new(j.X.Scale, math.clamp(j.X.Offset + O.X, f.MinSize.X, math.huge), j.Y.Scale, math.clamp(j.Y.Offset + O.Y, f.MinSize.Y, math.huge))
    if V then
        f:SafeCallback(V)
    end
end
end))
end
f.MakeCover = function(L, L, X)
    local V, K = Q[X] or{0, 0}, v[X] or{1, 0.5}
    return(F("Frame", {AnchorPoint = Vector2.new(V[1], V[2]), BackgroundColor3 = L.BackgroundColor3, Position = UDim2.fromScale(V[1], V[2]), Size = UDim2.fromScale(K[1], K[2]), Parent = L}))
end
f.MakeLine = function(L, L, Q)
    return(F("Frame", {AnchorPoint = Q.AnchorPoint or Vector2.zero, BackgroundColor3 = "OutlineColor", Position = Q.Position, Size = Q.Size, ZIndex = Q.ZIndex or L.ZIndex, Parent = L}))
end
f.AddOutline = function(L, L)
    return F("UIStroke", {Color = "OutlineColor", Thickness = 1, ZIndex = 2, Parent = L}), (F("UIStroke", {Color = "DarkColor", Thickness = 0, ZIndex = 1, Parent = L}))
end
f.AddBlank = function(L, L, Q)
    return F("Frame", {BackgroundTransparency = 1, Size = Q or(UDim2.fromScale(0, 0)), Parent = L})
end
f.MakeOutline = function(L, Q, v, X)
    warn("Obsidian is deprecated, please use Obsidian instead.")
    L = F("Frame", {BackgroundColor3 = "DarkColor", Position = UDim2.fromOffset(- 2, -2), Size = UDim2.new(1, 4, 1, 4), ZIndex = X, Parent = Q})
    Q = F("Frame", {BackgroundColor3 = "OutlineColor", Position = UDim2.fromOffset(1, 1), Size = UDim2.new(1, -2, 1, -2), ZIndex = X, Parent = L})
    if v and v > 0 then
        F("UICorner", {CornerRadius = UDim.new(0, v + 1), Parent = L})
        F("UICorner", {CornerRadius = UDim.new(0, v), Parent = Q})
    end
    return L, Q
end
f.AddDraggableLabel = function(L, L)
    local Q, v = {}, F("TextLabel", {AutomaticSize = Enum.AutomaticSize.XY, BackgroundColor3 = "BackgroundColor", Size = UDim2.fromOffset(0, 0), Position = UDim2.fromOffset(6, 6), Text = L, TextSize = 15, ZIndex = 10, Parent = b})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = v})
    F("UIPadding", {PaddingBottom = UDim.new(0, 6), PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12), PaddingTop = UDim.new(0, 6), Parent = v})
    table.insert(f.Scales, F("UIScale", {Parent = v}))
    f:AddOutline(v)
    f:MakeDraggable(v, v, true)
    Q.Label = v
    Q.SetText = function(L, L)
        pcall(function()
            v.Text = L
        end)
    end
    Q.SetVisible = function(L, L)
        pcall(function()
            v.Visible = L
        end)
    end
    return Q
end
f.AddDraggableButton = function(L, L, Q, v)
    local X, V = {}, F("TextButton", {BackgroundColor3 = "BackgroundColor", Position = UDim2.fromOffset(6, 6), TextSize = 16, ZIndex = 10, Parent = b})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = V})
    if not v then
        table.insert(f.Scales, F("UIScale", {Parent = V}))
    end
    f:AddOutline(V)
    V.MouseButton1Click:Connect(function()
        f:SafeCallback(Q, X)
    end)
    f:MakeDraggable(V, V, true)
    X.Button = V
    X.SetText = function(Q, Q)
        local v, K = f:GetTextBounds(Q, f.Scheme.Font, 16)
        V.Text = Q
        V.Size = UDim2.fromOffset(v * 2, K * 2)
    end
    X:SetText(L)
    return X
end
f.AddDraggableMenu = function(L, Q)
    L = F("Frame", {AutomaticSize = Enum.AutomaticSize.XY, BackgroundColor3 = "BackgroundColor", Position = UDim2.fromOffset(6, 6), Size = UDim2.fromOffset(0, 0), ZIndex = 10, Parent = b})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = L})
    table.insert(f.Scales, F("UIScale", {Parent = L}))
    f:AddOutline(L)
    f:MakeLine(L, {Position = UDim2.fromOffset(0, 34), Size = UDim2.new(1, 0, 0, 1)})
    local v = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 34), Text = Q, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left, Parent = L})
    F("UIPadding", {PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12), Parent = v})
    Q = F("Frame", {BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 35), Size = UDim2.new(1, 0, 1, -35), Parent = L})
    F("UIListLayout", {Padding = UDim.new(0, 7), Parent = Q})
    F("UIPadding", {PaddingBottom = UDim.new(0, 7), PaddingLeft = UDim.new(0, 7), PaddingRight = UDim.new(0, 7), PaddingTop = UDim.new(0, 7), Parent = Q})
    f:MakeDraggable(L, v, true)
    return L, Q
end
f.CreateFloatingPanel = function(L, Q)
    Q = Q or{}
    local v, X, V, K, j, O = Q.Title or "Panel", Q.Width or 580, Q.Height or 380, Q.Position or(UDim2.fromOffset(200, 100)), Q.SidebarWidth or 140, {Tabs = {}, ActiveTab = nil, TabOrder = {}}
    local Q = F("TextButton", {BackgroundColor3 = function()
        return f:GetBetterColor(f.Scheme.BackgroundColor, -1)
    end, Text = "", AutoButtonColor = false, BorderSizePixel = 0, Position = K, Size = UDim2.fromOffset(X, V), ClipsDescendants = true, Visible = false, ZIndex = 100, Parent = b})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = Q})
    table.insert(f.Scales, F("UIScale", {Parent = Q}))
    F("UIStroke", {Color = "AccentColor", Transparency = 0.7, Thickness = 1, Parent = Q})
    f:AddOutline(Q)
    V = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 36), Parent = Q})
    f:MakeDraggable(Q, V, true)
    f:MakeLine(Q, {Position = UDim2.fromOffset(0, 36), Size = UDim2.new(1, 0, 0, 1)})
    X = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(0, j, 1, 0), Parent = V})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Center, VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 6), Parent = X})
    local K = F("TextLabel", {BackgroundTransparency = 1, AutomaticSize = Enum.AutomaticSize.X, Size = UDim2.new(0, 0, 1, 0), Text = v, TextSize = 15, Parent = X})
    f:AddToRegistry(K, {TextColor3 = "FontColor"})
    L = F("Frame", {AnchorPoint = Vector2.new(1, 0.5), BackgroundTransparency = 1, Position = UDim2.new(1, -8, 0.5, 0), Size = UDim2.new(1, -j -16, 0, 20), Parent = V})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Right, VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 4), Parent = L})
    local v = F("TextLabel", {BackgroundTransparency = 1, AutomaticSize = Enum.AutomaticSize.X, Size = UDim2.new(0, 0, 1, 0), Text = "", TextSize = 12, TextTransparency = 0.5, Parent = L})
    f:AddToRegistry(v, {TextColor3 = "FontColor"})
    local X = F("TextButton", {BackgroundTransparency = 1, Size = UDim2.fromOffset(36, 36), AnchorPoint = Vector2.new(1, 0), Position = UDim2.new(1, 0, 0, 0), Text = "X", TextSize = 14, Parent = V})
    f:AddToRegistry(X, {TextColor3 = "FontColor"})
    X.MouseEnter:Connect(function()
        pcall(function()
            X.TextColor3 = f.Scheme.RedColor
        end)
    end)
    X.MouseLeave:Connect(function()
        pcall(function()
            X.TextColor3 = f.Scheme.FontColor
        end)
    end)
    X.MouseButton1Click:Connect(function()
        Q.Visible = false
    end)
    L = F("Frame", {BackgroundColor3 = "BackgroundColor", Position = UDim2.fromOffset(0, 37), Size = UDim2.new(0, j, 1, -37), Parent = Q})
    local X = F("ScrollingFrame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), CanvasSize = UDim2.fromScale(0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y, ScrollBarThickness = 2, ScrollBarImageTransparency = 0.6, Parent = L})
    F("UIListLayout", {Padding = UDim.new(0, 2), Parent = X})
    F("UIPadding", {PaddingTop = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6), Parent = X})
    F("Frame", {BackgroundColor3 = "OutlineColor", Position = UDim2.new(0, j, 0, 37), Size = UDim2.new(0, 1, 1, -37), Parent = Q})
    local L = F("ScrollingFrame", {BackgroundTransparency = 1, Position = UDim2.fromOffset(j + 1, 37), Size = UDim2.new(1, -j -1, 1, -37), CanvasSize = UDim2.fromScale(0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y, ScrollBarThickness = 3, ScrollBarImageTransparency = 0.4, Parent = Q})
    f:AddToRegistry(L, {ScrollBarImageColor3 = "AccentColor"})
    F("UIPadding", {PaddingTop = UDim.new(0, 10), PaddingBottom = UDim.new(0, 10), PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12), Parent = L})
    O.Frame = Q
    O.SetVisible = function(V, V)
        Q.Visible = V
    end
    O.Toggle = function(V)
        Q.Visible = not Q.Visible
    end
    O.SetTitle = function(Q, Q)
        K.Text = Q
    end
    O.AddTab = function(Q, V, K)
        local j = F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 1, AutoButtonColor = false, BorderSizePixel = 0, Size = UDim2.new(1, 0, 0, 28), Text = "", Parent = X})
        F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius / 2), Parent = j})
        F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 6), Parent = j})
        F("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), Parent = j})
        Q = K and(f:GetIcon(K))
        if Q then
            K = F("ImageLabel", {BackgroundTransparency = 1, Size = UDim2.fromOffset(14, 14), Image = Q.Url or Q, Parent = j})
            if Q.ImageRectOffset then
                K.ImageRectOffset = Q.ImageRectOffset
            end
            if Q.ImageRectSize then
                K.ImageRectSize = Q.ImageRectSize
            end
            f:AddToRegistry(K, {ImageColor3 = "FontColor"})
        end
        Q = F("TextLabel", {BackgroundTransparency = 1, AutomaticSize = Enum.AutomaticSize.X, Size = UDim2.new(0, 0, 1, 0), Text = V, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, Parent = j})
        f:AddToRegistry(Q, {TextColor3 = "FontColor"})
        K = F("Frame", {BackgroundColor3 = "AccentColor", Position = UDim2.fromOffset(0, 0), Size = UDim2.new(0, 3, 1, 0), Visible = false, Parent = j})
        F("UICorner", {CornerRadius = UDim.new(0, 2), Parent = K})
        local X = F("ScrollingFrame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), CanvasSize = UDim2.fromScale(0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y, ScrollBarThickness = 3, ScrollBarImageTransparency = 0.4, BorderSizePixel = 0, Visible = false, Parent = L})
        f:AddToRegistry(X, {ScrollBarImageColor3 = "AccentColor"})
        local L = F("Frame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), AutomaticSize = Enum.AutomaticSize.Y, Parent = X})
        F("UIListLayout", {Padding = UDim.new(0, 8), Parent = L})
        F("UIPadding", {PaddingBottom = UDim.new(0, 7), PaddingLeft = UDim.new(0, 7), PaddingRight = UDim.new(0, 7), PaddingTop = UDim.new(0, 7), Parent = L})
        local m = {Container = L, Holder = L, Tab = {}, DependencyBoxes = {}, Elements = {}, Resize = function(k)
        end
        }
        setmetatable(m, f.BaseGroupbox)
        L = {Name = V, Button = j, Label = Q, AccentLine = K, Scroll = X, Groupbox = m, Container = X}
        j.MouseEnter:Connect(function()
            if O.ActiveTab ~= V then j.BackgroundTransparency = 0.7
        end
    end)
    j.MouseLeave:Connect(function()
        if O.ActiveTab ~= V then j.BackgroundTransparency = 1
    end
end)
j.MouseButton1Click:Connect(function()
    O:SwitchTab(V)
end)
table.insert(O.TabOrder, V)
O.Tabs[V] = L
if not O.ActiveTab then
    O:SwitchTab(V)
end
return m
end
O.SwitchTab = function(L, Q)
    for X, V in pairs(O.Tabs) do
        L = X == Q
        V.Container.Visible = L
        V.AccentLine.Visible = L
        V.Button.BackgroundTransparency = L and 0 or 1
        V.Label.TextTransparency = L and 0 or 0.4
    end
    v.Text = Q
    O.ActiveTab = Q
end
return O
end
do
    local L = f:AddDraggableLabel("")
    L:SetVisible(false)
    f.SetWatermark = function(Q, Q)
        warn("Watermark is deprecated, please use Library instead.")
        L:SetText(Q)
    end
    f.SetWatermarkVisibility = function(Q, Q)
        warn("Watermark is deprecated, please use Library instead.")
        L:SetVisible(Q)
    end
end
local L
local function Q()
    return f.Window and f.Window.Frame
end
f.AddContextMenu = function(v, v, X, V, K, j)
    local O
    if K then
        O = F("ScrollingFrame", {AutomaticCanvasSize = K == 2 and Enum.AutomaticSize.Y or Enum.AutomaticSize.None, AutomaticSize = K == 1 and Enum.AutomaticSize.Y or Enum.AutomaticSize.None, BackgroundColor3 = "BackgroundColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, BottomImage = "rbxasset://textures/ui/Scroll/scroll-middle.png", CanvasSize = UDim2.fromOffset(0, 0), ClipsDescendants = true, ScrollBarImageColor3 = "OutlineColor", ScrollBarThickness = K == 2 and 2 or 0, Size = typeof(X) == "function" and(X()) or X, TopImage = "rbxasset://textures/ui/Scroll/scroll-middle.png", Visible = false, ZIndex = 50, Parent = b})
    else
        O = F("Frame", {BackgroundColor3 = "BackgroundColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, ClipsDescendants = true, Size = typeof(X) == "function" and(X()) or X, Visible = false, ZIndex = 50, Parent = b})
    end
    table.insert(f.Scales, F("UIScale", {Parent = O}))
    local m = {Active = false, Holder = v, Menu = O, List = nil, Signal = nil, ScrollConns = {}, Size = X}
    if K then
        m.List = F("UIListLayout", {Parent = O})
    end
    local function X()
        if typeof(V) == "function" then local K = V()
        return K[1] or 0, K[2] or 0
    end
    return V[1], V[2]
end
local function V()
    local K, k = X()
    local X, n = v.AbsolutePosition.X + K, v.AbsolutePosition.Y + k
    O.Size = typeof(m.Size) == "function" and(m.Size()) or m.Size
    k = Q()
    if k and k.Parent then
        local K, E, S, G = k.AbsolutePosition, k.AbsoluteSize, O.AbsoluteSize.X, O.AbsoluteSize.Y
        S, G = if S <= 0 and O.Size.X.Offset > 0 then O.Size.X.Offset * f.DPIScale else S, if G <= 0 and O.Size.Y.Offset > 0 then O.Size.Y.Offset * f.DPIScale else G
        local k, y = n + G, K.Y
        n, X = if G > 0 then(math.clamp(if k > y + E.Y then v.AbsolutePosition.Y -G -2 else n, K.Y, math.max(K.Y, K.Y + E.Y -G))) else
        if k > y + E.Y then
            v.AbsolutePosition.Y -G -2
        else
            n, if S > 0 then(math.clamp(X, K.X, math.max(K.X, K.X + E.X -S))) else X
        end
        O.Position = UDim2.fromOffset(math.floor(X), math.floor(n))
    end
    m.UnbindScrollClose = function(X)
        for X, X in ipairs(m.ScrollConns) do
            pcall(function()
                X:Disconnect()
            end)
        end
        table.clear(m.ScrollConns)
    end
    m.BindScrollClose = function(X)
        m:UnbindScrollClose()
        local X, K = v.Parent, Q()
        while X and X ~= b do
            if X:IsA("ScrollingFrame") then
                table.insert(m.ScrollConns, X:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
                    m:Close()
                end))
            end
            if X == K then
                break
            end
            X = X.Parent
        end
    end
    m.Open = function(Q)
        if L == m then return
    elseif L then
        L:Close()
    end
    L = m
    m.Active = true
    V()
    if typeof(j) == "function" then
        f:SafeCallback(j, true)
    end
    O.Visible = true
    m:BindScrollClose()
    m.Signal = v:GetPropertyChangedSignal("AbsolutePosition"):Connect(V)
end
m.Close = function(Q)
    if L ~= m then return
end
O.Visible = false
if m.Signal then
    m.Signal:Disconnect()
    m.Signal = nil
end
m:UnbindScrollClose()
m.Active = false
L = nil
if typeof(j) == "function" then
    f:SafeCallback(j, false)
end
end
m.Toggle = function(Q)
    if m.Active then m:Close()
else
    m:Open()
end
end
m.SetSize = function(Q, Q)
    m.Size = Q
    O.Size = typeof(Q) == "function" and(Q()) or Q
    if m.Active then
        V()
    end
end
m.Destroy = function(Q)
    m:Close()
    m:UnbindScrollClose()
    Q = L
    if Q == m then
        L = nil
    end
    for v = #f.Scales, 1, -1 do
        Q = f.Scales[v]
        if Q and Q.Parent == O then
            table.remove(f.Scales, v)
        end
    end
    pcall(function()
        O:Destroy()
    end)
end
return m
end
f:GiveSignal(U.InputBegan:Connect(function(Q)
    if f.Unloaded then return
end
if a(Q, true) then
    local v = Q.Position
    if L and not(f:MouseIsOverFrame(L.Menu, v) or(f:MouseIsOverFrame(L.Holder, v))) then
        L:Close()
    end
end
end))
local Q = F("TextLabel", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = "BackgroundColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, TextSize = 14, TextWrapped = true, Visible = false, ZIndex = 20, Parent = b})
F("UIPadding", {PaddingBottom = UDim.new(0, 2), PaddingLeft = UDim.new(0, 4), PaddingRight = UDim.new(0, 4), PaddingTop = UDim.new(0, 2), Parent = Q})
table.insert(f.Scales, (F("UIScale", {Parent = Q})))
Q:GetPropertyChangedSignal("AbsolutePosition"):Connect(function()
    if f.Unloaded then return
end
local v = f:GetTextBounds(Q.Text, Q.FontFace, Q.TextSize, (workspace.CurrentCamera.ViewportSize.X -Q.AbsolutePosition.X -8) / f.DPIScale)
Q.Size = UDim2.fromOffset(v + 8)
end)
local v
f.AddTooltip = function(X, X, V, K)
    local j = {Disabled = false, Hovering = false, Signals = {}}
    local function O()
        if v == K or L and(f:MouseIsOverFrame(L.Menu, C)) or j.Disabled and typeof(V) ~= "string" or not j.Disabled and typeof(X) ~= "string" then return
    end
    v = K
    Q.Text = j.Disabled and V or X
    Q.Visible = true
    while f.Toggled and(f:MouseIsOverFrame(K, C)) and not(L and(f:MouseIsOverFrame(L.Menu, C))) do
        Q.Position = UDim2.fromOffset(C.X + (f.ShowCustomCursor and 8 or 14), C.Y + (f.ShowCustomCursor and 8 or 12))
        q.RenderStepped:Wait()
    end
    Q.Visible = false
    v = nil
end
local function L(X)
    local V = typeof(X)
    if X and(V == "RBXScriptConnection" or V == "RBXScriptSignal") then
        table.insert(j.Signals, X)
    end
    return X
end
L(K.MouseEnter:Connect(O))
L(K.MouseMoved:Connect(O))
L(K.MouseLeave:Connect(function()
    if v ~= K then return
end
Q.Visible = false
v = nil
end))
j.Destroy = function(L)
    for X = #j.Signals, 1, -1 do
        L = table.remove(j.Signals, X)
        if L and L.Connected then
            L:Disconnect()
        end
    end
    if v == K then
        if Q then
            Q.Visible = false
        end
        v = nil
    end
end
table.insert(P, Q)
return j
end
f.OnUnload = function(L, L)
    table.insert(f.UnloadSignals, L)
end
f.Unload = function(L)
    for v = #f.Signals, 1, -1 do
        L = table.remove(f.Signals, v)
        if L and L.Connected then
            L:Disconnect()
        end
    end
    for L, L in pairs(f.UnloadSignals) do
        f:SafeCallback(L)
    end
    for L, L in pairs(P) do
        f:SafeCallback(L.Destroy, L)
    end
    for L, L in pairs(game:GetService("Lighting"):GetChildren()) do
        if L:IsA("BlurEffect") then
            L:Destroy()
        end
    end
    f.Unloaded = true
    b:Destroy()
    R().YabujinLibrary = nil
end
local L, P, v, X, V = f:GetIcon("check"), f:GetIcon("chevron-up"), f:GetIcon("move-diagonal-2"), f:GetIcon("key"), f:GetIcon("move")
f.SetIconModule = function(K, K)
    I, x = true, K
    L = f:GetIcon("check")
    P = f:GetIcon("chevron-up")
    v = f:GetIcon("move-diagonal-2")
    X = f:GetIcon("key")
    V = f:GetIcon("move")
end
local I = {}
do
    local x, K = {AddKeyPicker = function(j, O, m)
        m = f:Validate(m, l.KeyPicker)
        local k = j
        local n, E = k.TextLabel, {Text = m.Text, Value = m.Default, Modifiers = m.DefaultModifiers, DisplayValue = m.Default, Toggled = false, Mode = m.Mode, SyncToggleState = m.SyncToggleState, Callback = m.Callback, ChangedCallback = m.ChangedCallback, Changed = m.Changed, Clicked = m.Clicked, Type = "KeyPicker"}
        if E.Mode == "Press" then
            assert(k.Type == "Label", "KeyPicker with the mode 'Press' can be only applied on Labels.")
            E.SyncToggleState = false
            m.Modes = {"Press"}
            m.Mode = "Press"
        end
        if E.SyncToggleState then
            m.Modes = {"Toggle", "Hold"}
            if not table.find(m.Modes, m.Mode) then
                m.Mode = "Toggle"
            end
        end
        local S, G, y, i, T = false, {MB1 = Enum.UserInputType.MouseButton1, MB2 = Enum.UserInputType.MouseButton2, MB3 = Enum.UserInputType.MouseButton3}, {[Enum.UserInputType.MouseButton1] = "MB1", [Enum.UserInputType.MouseButton2] = "MB2", [Enum.UserInputType.MouseButton3] = "MB3"}, {LAlt = Enum.KeyCode.LeftAlt, RAlt = Enum.KeyCode.RightAlt, LCtrl = Enum.KeyCode.LeftControl, RCtrl = Enum.KeyCode.RightControl, LShift = Enum.KeyCode.LeftShift, RShift = Enum.KeyCode.RightShift, Tab = Enum.KeyCode.Tab, CapsLock = Enum.KeyCode.CapsLock}, {[Enum.KeyCode.LeftAlt] = "LAlt", [Enum.KeyCode.RightAlt] = "RAlt", [Enum.KeyCode.LeftControl] = "LCtrl", [Enum.KeyCode.RightControl] = "RCtrl", [Enum.KeyCode.LeftShift] = "LShift", [Enum.KeyCode.RightShift] = "RShift", [Enum.KeyCode.Tab] = "Tab", [Enum.KeyCode.CapsLock] = "CapsLock"}
        local function MJ(bJ)
            return bJ.UserInputType == Enum.UserInputType.Keyboard and T[bJ.KeyCode] ~= nil
        end
        local function bJ()
            local JJ = {}
            for NJ, AJ in pairs(i) do
                if not table.find(JJ, NJ) and(U:IsKeyDown(AJ)) then
                    table.insert(JJ, NJ)
                end
            end
            return JJ
        end
        local function JJ(NJ)
            if not(typeof(NJ) == "table" and p(NJ) > 0) then return true
        end
        local AJ, qJ = bJ(), true
        for bJ, bJ in pairs(NJ) do
            if not table.find(AJ, bJ) then
                qJ = false
                break
            end
        end
        return qJ
    end
    local function bJ(NJ)
        if not NJ then return false
    end
    if y[NJ.UserInputType] ~= nil then
        return U:IsMouseButtonPressed(NJ.UserInputType) and not U:GetFocusedTextBox()
    elseif NJ.UserInputType == Enum.UserInputType.Keyboard then
        return U:IsKeyDown(NJ.KeyCode) and not U:GetFocusedTextBox()
    else
        return false
    end
end
local function NJ(AJ)
    local qJ = {}
    for WJ, WJ in pairs(AJ) do
        table.insert(qJ, i[WJ])
    end
    return qJ
end
local function AJ(qJ)
    if typeof(qJ) ~= "table" then return{}
end
local WJ = {}
for UJ, UJ in pairs(qJ) do
    if i[UJ] then
        table.insert(WJ, UJ)
    end
end
return WJ
end
E.Modifiers = AJ(E.Modifiers)
local i, qJ = F("TextButton", {BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Size = UDim2.fromOffset(18, 18), Text = E.Value, TextSize = 14, Parent = n}), {Normal = E.Mode ~= "Toggle"}
do
    local WJ = F("TextButton", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 16), Text = "", Visible = not m.NoUI, Parent = f.KeybindContainer})
    local UJ, LJ = F("TextLabel", {AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Size = UDim2.fromScale(0, 1), Text = "", TextSize = 14, TextTransparency = 0.5, Parent = WJ}), F("Frame", {AnchorPoint = Vector2.new(0, 0.5), BackgroundColor3 = "MainColor", Position = UDim2.fromScale(0, 0.5), Size = UDim2.fromOffset(14, 14), SizeConstraint = Enum.SizeConstraint.RelativeYY, Parent = WJ})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius / 2), Parent = LJ})
    F("UIStroke", {Color = "OutlineColor", Parent = LJ})
    local hJ = F("ImageLabel", {Image = L and L.Url or "", ImageColor3 = "FontColor", ImageRectOffset = L and L.ImageRectOffset or Vector2.zero, ImageRectSize = L and L.ImageRectSize or Vector2.zero, ImageTransparency = 1, Position = UDim2.fromOffset(2, 2), Size = UDim2.new(1, -4, 1, -4), Parent = LJ})
    qJ.Display = function(BJ, BJ)
        UJ.TextTransparency = BJ and 0 or 0.5
        hJ.ImageTransparency = BJ and 0 or 1
    end
    qJ.SetText = function(hJ, hJ)
        UJ.Text = hJ
    end
    qJ.SetVisibility = function(hJ, hJ)
        WJ.Visible = hJ
    end
    qJ.SetNormal = function(hJ, BJ)
        qJ.Normal = BJ
        hJ = not BJ
        WJ.Active = hJ
        UJ.Position = BJ and(UDim2.fromOffset(0, 0)) or(UDim2.fromOffset(22, 0))
        LJ.Visible = hJ
    end
    E.DoClick = function(...)
    end
    WJ.MouseButton1Click:Connect(function()
        if qJ.Normal then return
    end
    E.Toggled = not E.Toggled
    E:DoClick()
end)
qJ.Holder = WJ
qJ.Label = UJ
qJ.Checkbox = LJ
qJ.Loaded = true
table.insert(f.KeybindToggles, qJ)
end
local WJ = f:AddContextMenu(i, UDim2.fromOffset(62, 0), function()
    return{i.AbsoluteSize.X + 1.5, 0.5}
end, 1)
E.Menu = WJ
local UJ = {}
for LJ, LJ in pairs(m.Modes) do
    local hJ, BJ = {}, F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 21), Text = LJ, TextSize = 14, TextTransparency = 0.5, Parent = WJ.Menu})
    hJ.Select = function(RJ)
        for RJ, RJ in pairs(UJ) do
            RJ:Deselect()
        end
        E.Mode = LJ
        BJ.BackgroundTransparency = 0
        BJ.TextTransparency = 0
        WJ:Close()
    end
    hJ.Deselect = function(RJ)
        E.Mode = nil
        BJ.BackgroundTransparency = 1
        BJ.TextTransparency = 0.5
    end
    BJ.MouseButton1Click:Connect(function()
        hJ:Select()
    end)
    if E.Mode == LJ then
        hJ:Select()
    end
    UJ[LJ] = hJ
end
E.Display = function(LJ, LJ)
    if f.Unloaded then return
end
local hJ, BJ = f:GetTextBounds(LJ or E.DisplayValue, i.FontFace, i.TextSize, n.AbsoluteSize.X)
i.Text = LJ or E.DisplayValue
i.Size = UDim2.fromOffset(hJ + 9, BJ + 4)
end
E.Update = function(n)
    E:Display()
    if m.NoUI then
        return
    end
    if E.Mode == "Toggle" and k.Type == "Toggle" and k.Disabled then
        qJ:SetVisibility(false)
        return
    end
    local n, LJ = E:GetState(), f.ShowToggleFrameInKeybinds and E.Mode == "Toggle"
    if E.SyncToggleState and k.Value ~= n then
        k:SetValue(n)
    end
    if qJ.Loaded then
        if LJ then
            qJ:SetNormal(false)
        else
            qJ:SetNormal(true)
        end
        qJ:SetText(("[%s] %s (%s)"):format(E.DisplayValue, E.Text, E.Mode))
        qJ:SetVisibility(true)
        qJ:Display(n)
    end
end
E.GetState = function(n)
    if E.Mode == "Always" then return true
elseif E.Mode == "Hold" then
    n = E.Value
    if n == "None" then
        return false
    end
    if not JJ(E.Modifiers) then
        return false
    end
    if G[n] ~= nil then
        return U:IsMouseButtonPressed(G[n]) and not U:GetFocusedTextBox()
    else
        return U:IsKeyDown(Enum.KeyCode[n]) and not U:GetFocusedTextBox()
    end
else
    return E.Toggled
end
end
E.OnChanged = function(n, n)
    E.Changed = n
end
E.OnClick = function(n, n)
    E.Clicked = n
end
E.DoClick = function(n)
    if E.Mode == "Press" then
    if E.Toggled and m.WaitForCallback == true then
        return
    end
    E.Toggled = true
end
f:SafeCallback(E.Callback, E.Toggled)
f:SafeCallback(E.Clicked, E.Toggled)
if E.Mode == "Press" then
    E.Toggled = false
end
end
E.SetValue = function(m, n)
    local LJ, hJ, BJ = n[1], n[2], n[3]
    m, n = pcall(function()
        if LJ == "None" then LJ = nil
        return nil
    end
    if G[LJ] == nil then
        return Enum.KeyCode[LJ]
    end
    return G[LJ]
end)
if LJ == nil then
    E.Value = "None"
elseif m then
    E.Value = LJ
else
    E.Value = "Unknown"
end
E.Modifiers = AJ(typeof(BJ) == "table" and BJ or E.Modifiers)
E.DisplayValue = p(E.Modifiers) > 0 and table.concat(E.Modifiers, " + ") .. " + " .. E.Value or E.Value
if UJ[hJ] then
    UJ[hJ]:Select()
end
m = NJ(E.Modifiers)
f:SafeCallback(E.ChangedCallback, n, m)
f:SafeCallback(E.Changed, n, m)
E:Update()
end
E.SetText = function(m, m)
    qJ:SetText(m)
    E:Update()
end
i.MouseButton1Click:Connect(function()
    if S then return
end
S = true
i.Text = "..."
i.Size = UDim2.fromOffset(29, 18)
local m
local n = {}
local function G()
    m = U.InputBegan:Wait()
    return U:GetFocusedTextBox() ~= nil
end
repeat
    task.wait()
    i.Text = "..."
    i.Size = UDim2.fromOffset(29, 18)
    if G() then
        S = false
        E:Update()
        return
    end
    if m.KeyCode == Enum.KeyCode.Escape then
        break
    end
    if MJ(m) then
        local NJ = false
        repeat
            task.wait()
            if U:IsKeyDown(m.KeyCode) then
                task.wait(0.075)
                if U:IsKeyDown(m.KeyCode) then
                    if not table.find(n, T[m.KeyCode]) then
                        n[#n + 1] = T[m.KeyCode]
                        E:Display(table.concat(n, " + ") .. " + ...")
                    end
                    local AJ = NJ
                    if G() then
                        NJ = true
                        break
                    end
                    if m.KeyCode == Enum.KeyCode.Escape then
                        NJ = AJ
                        break
                    end
                    if not MJ(m) then
                        NJ = AJ
                        break
                    end
                elseif not table.find(n, T[m.KeyCode]) then
                    break
                end
            end
            until false
            if NJ then
                S = false
                E:Update()
                return
            end
        end
        break
        until false
        local G, T = if y[m.UserInputType] ~= nil then y[m.UserInputType] else
        if m.UserInputType == Enum.UserInputType.Keyboard then
            m.KeyCode == Enum.KeyCode.Escape and "None" or m.KeyCode.Name
        else
            "Unknown", {"Touch", "MB1", "MB2", "MB3", "W", "A", "S", "D", "Space", "LeftShift", "LeftControl"}
            for MJ, MJ in pairs(T) do
                if G == MJ then
                    f:Notify("Cant use " .. MJ .. " as keybind", 3)
                    S = false
                    E:Update()
                    return
                end
            end
            n = (m.KeyCode == Enum.KeyCode.Escape or G == "Unknown") and{} or n
            E.Toggled = false
            E:SetValue({G, E.Mode, n})
            repeat
                task.wait()
                until not bJ(m) or(U:GetFocusedTextBox())
                S = false
            end)
            i.MouseButton2Click:Connect(WJ.Toggle)
            f:GiveSignal(U.InputBegan:Connect(function(m)
                if f.Unloaded then return
            end
            if E.Mode == "Always" or E.Value == "Unknown" or E.Value == "None" or S or(U:GetFocusedTextBox()) then
                return
            end
            local n = E.Value
            local G = if n and JJ(E.Modifiers) == true and(y[m.UserInputType] == n or m.UserInputType == Enum.UserInputType.Keyboard and m.KeyCode.Name == n) then true else false
            if E.Mode == "Toggle" then
                if G then
                    E.Toggled = not E.Toggled
                    E:DoClick()
                end
            elseif E.Mode == "Press" then
                if G then
                    E:DoClick()
                end
            end
            E:Update()
        end))
        f:GiveSignal(U.InputEnded:Connect(function()
            if f.Unloaded then return
        end
        if E.Value == "Unknown" or E.Value == "None" or S or(U:GetFocusedTextBox()) then
            return
        end
        E:Update()
    end))
    E:Update()
    if k.Addons then
        table.insert(k.Addons, E)
    end
    E.Default = E.Value
    E.DefaultModifiers = table.clone(E.Modifiers or{})
    z[O] = E
    return j
end
}, {}
for j = 0, 1, 0.1 do
    table.insert(K, ColorSequenceKeypoint.new(j, Color3.fromHSV(j, 1, 1)))
end
x.AddColorPicker = function(j, O, m)
    m = f:Validate(m, l.ColorPicker)
    local k, n = j.TextLabel, {Value = m.Default, Transparency = m.Transparency or 0, Title = m.Title, Callback = m.Callback, Changed = m.Changed, Type = "ColorPicker"}
    n.Hue, n.Sat, n.Vib = n.Value:ToHSV()
    local E = F("TextButton", {BackgroundColor3 = n.Value, BorderColor3 = f:GetDarkerColor(n.Value), BorderSizePixel = 1, Size = UDim2.fromOffset(18, 18), Text = "", Parent = k})
    local S, G = F("ImageLabel", {Image = Y.GetAsset("TransparencyTexture"), ImageTransparency = 1 -n.Transparency, ScaleType = Enum.ScaleType.Tile, Size = UDim2.fromScale(1, 1), TileSize = UDim2.fromOffset(9, 9), Parent = E}), f:AddContextMenu(E, UDim2.fromOffset(m.Transparency and 256 or 234, 0), function()
        return{0.5, E.AbsoluteSize.Y + 1.5}
    end, 1)
    G.List.Padding = UDim.new(0, 8)
    n.ColorMenu = G
    F("UIPadding", {PaddingBottom = UDim.new(0, 6), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6), PaddingTop = UDim.new(0, 6), Parent = G.Menu})
    if typeof(n.Title) == "string" then
        F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 8), Text = n.Title, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = G.Menu})
    end
    k = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 200), Parent = G.Menu})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6), Parent = k})
    local y = F("ImageButton", {BackgroundColor3 = n.Value, Image = Y.GetAsset("SaturationMap"), Size = UDim2.fromOffset(200, 200), Parent = k})
    local i = F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "WhiteColor", Size = UDim2.fromOffset(6, 6), Parent = y})
    F("UICorner", {CornerRadius = UDim.new(1, 0), Parent = i})
    F("UIStroke", {Color = "DarkColor", Parent = i})
    local T = F("TextButton", {Size = UDim2.fromOffset(16, 200), Text = "", Parent = k})
    F("UIGradient", {Color = ColorSequence.new(K), Rotation = 90, Parent = T})
    local K = F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "WhiteColor", BorderColor3 = "DarkColor", BorderSizePixel = 1, Position = UDim2.fromScale(0.5, n.Hue), Size = UDim2.new(1, 2, 0, 1), Parent = T})
    local MJ, bJ, JJ
    if m.Transparency then
        MJ = F("ImageButton", {Image = Y.GetAsset("TransparencyTexture"), ScaleType = Enum.ScaleType.Tile, Size = UDim2.fromOffset(16, 200), TileSize = UDim2.fromOffset(8, 8), Parent = k})
        bJ = F("Frame", {BackgroundColor3 = n.Value, Size = UDim2.fromScale(1, 1), Parent = MJ})
        F("UIGradient", {Rotation = 90, Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(1, 1)}), Parent = bJ})
        JJ = F("Frame", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "WhiteColor", BorderColor3 = "DarkColor", BorderSizePixel = 1, Position = UDim2.fromScale(0.5, n.Transparency), Size = UDim2.new(1, 2, 0, 1), Parent = MJ})
    end
    k = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 20), Parent = G.Menu})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalFlex = Enum.UIFlexAlignment.Fill, Padding = UDim.new(0, 8), Parent = k})
    local Y, NJ, AJ = F("TextBox", {BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, ClearTextOnFocus = false, Size = UDim2.fromScale(1, 1), Text = "#??????", TextSize = 14, Parent = k}), F("TextBox", {BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, ClearTextOnFocus = false, Size = UDim2.fromScale(1, 1), Text = "?, ?, ?", TextSize = 14, Parent = k}), f:AddContextMenu(E, UDim2.fromOffset(93, 0), function()
        return{E.AbsoluteSize.X + 1.5, 0.5}
    end, 1)
    n.ContextMenu = AJ
    local function k(qJ, WJ)
        F("TextButton", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 21), Text = qJ, TextSize = 14, Parent = AJ.Menu}).MouseButton1Click:Connect(function()
            f:SafeCallback(WJ)
            AJ:Close()
        end)
    end
    k("Copy color", function()
        f.CopiedColor = {n.Value, n.Transparency}
    end)
    n.SetValueRGB = function(...)
    end
    k("Paste color", function()
        n:SetValueRGB(f.CopiedColor[1], f.CopiedColor[2])
    end)
    if s then
        k("Copy Hex", function()
            s(tostring(n.Value:ToHex()))
        end)
        k("Copy RGB", function()
            s(table.concat({math.floor(n.Value.R * 255), math.floor(n.Value.G * 255), math.floor(n.Value.B * 255)}, ", "))
        end)
    end
    n.SetHSVFromRGB = function(s, s)
        n.Hue, n.Sat, n.Vib = s:ToHSV()
    end
    n.Display = function(s)
        if f.Unloaded then return
    end
    n.Value = Color3.fromHSV(n.Hue, n.Sat, n.Vib)
    E.BackgroundColor3 = n.Value
    E.BorderColor3 = f:GetDarkerColor(n.Value)
    S.ImageTransparency = 1 -n.Transparency
    y.BackgroundColor3 = Color3.fromHSV(n.Hue, 1, 1)
    if bJ then
        bJ.BackgroundColor3 = n.Value
    end
    i.Position = UDim2.fromScale(n.Sat, 1 -n.Vib)
    K.Position = UDim2.fromScale(0.5, n.Hue)
    if JJ then
        JJ.Position = UDim2.fromScale(0.5, n.Transparency)
    end
    Y.Text = "#" .. n.Value:ToHex()
    NJ.Text = table.concat({math.floor(n.Value.R * 255), math.floor(n.Value.G * 255), math.floor(n.Value.B * 255)}, ", ")
end
n.Update = function(s)
    n:Display()
    f:SafeCallback(n.Callback, n.Value)
    f:SafeCallback(n.Changed, n.Value)
end
n.OnChanged = function(s, s)
    n.Changed = s
end
n.SetValue = function(s, K, k)
    if typeof(K) == "Color3" then n:SetValueRGB(K, k)
    return
end
s = Color3.fromHSV(K[1], K[2], K[3])
n.Transparency = m.Transparency and k or 0
n:SetHSVFromRGB(s)
n:Update()
end
n.SetValueRGB = function(s, s, K)
    n.Transparency = m.Transparency and K or 0
    n:SetHSVFromRGB(s)
    n:Update()
end
E.MouseButton1Click:Connect(G.Toggle)
E.MouseButton2Click:Connect(AJ.Toggle)
y.InputBegan:Connect(function(s)
    while g(s) do
        local s = y.AbsolutePosition.X
        local K = s + y.AbsoluteSize.X
        local m, k = math.clamp(C.X, s, K), y.AbsolutePosition.Y
        local E = k + y.AbsoluteSize.Y
        local S, G, y = math.clamp(C.Y, k, E), n.Sat, n.Vib
        n.Sat = (m -s) / (K -s)
        n.Vib = 1 -(S -k) / (E -k)
        if n.Sat ~= G or n.Vib ~= y then
            n:Update()
        end
        q.RenderStepped:Wait()
    end
end)
T.InputBegan:Connect(function(s)
    while g(s) do
        local s = T.AbsolutePosition.Y
        local K = s + T.AbsoluteSize.Y
        local m, k = math.clamp(C.Y, s, K), n.Hue
        n.Hue = (m -s) / (K -s)
        if n.Hue ~= k then
            n:Update()
        end
        q.RenderStepped:Wait()
    end
end)
if MJ then
    MJ.InputBegan:Connect(function(s)
        while g(s) do
            local s, K = MJ.AbsolutePosition.Y, MJ.AbsolutePosition.Y + MJ.AbsoluteSize.Y
            local m, k = math.clamp(C.Y, s, K), n.Transparency
            n.Transparency = (m -s) / (K -s)
            if n.Transparency ~= k then
                n:Update()
            end
            q.RenderStepped:Wait()
        end
    end)
end
Y.FocusLost:Connect(function(s)
    if not s then return
end
local s, K = pcall(Color3.fromHex, Y.Text)
if s and typeof(K) == "Color3" then
    n.Hue, n.Sat, n.Vib = K:ToHSV()
end
n:Update()
end)
NJ.FocusLost:Connect(function(s)
    if not s then return
end
local s, Y, K = NJ.Text:match("(%d+),%s*(%d+),%s*(%d+)")
if s and Y and K then
    n:SetHSVFromRGB(Color3.fromRGB(s, Y, K))
end
n:Update()
end)
n:Display()
if j.Addons then
    table.insert(j.Addons, n)
end
n.Default = n.Value
z[O] = n
return j
end
x.AddInlineButton = function(s, Y)
    Y = if typeof(Y) ~= "table" then{Text = tostring(Y or "")} else Y
    local K = s.TextLabel
    assert(K, "AddInlineButton can only be applied on Toggles or Labels.")
    local j, O = {Text = Y.Text or "", Icon = Y.Icon, Func = Y.Func or Y.Callback or function()
    end, Tooltip = Y.Tooltip, DisabledTooltip = Y.DisabledTooltip, TooltipTable = nil, Risky = Y.Risky or false, Disabled = Y.Disabled or false, Visible = Y.Visible ~= false, Tween = nil, Type = "InlineButton"}, Y.Size or 16
    local m = F("TextButton", {Active = not j.Disabled, BackgroundColor3 = j.Disabled and "BackgroundColor" or "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Size = UDim2.fromOffset(O, O), Text = "", TextSize = 11, TextTransparency = j.Disabled and 0.8 or 0.35, Visible = j.Visible, Parent = K})
    local K, O = F("UIStroke", {Color = "OutlineColor", Transparency = j.Disabled and 0.5 or 0, Parent = m}), false
    if j.Icon then
        Y = f:GetIcon(j.Icon)
        if Y then
            F("ImageLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Image = Y.Url or "", ImageColor3 = "FontColor", ImageRectOffset = Y.ImageRectOffset or Vector2.zero, ImageRectSize = Y.ImageRectSize or Vector2.zero, ImageTransparency = j.Disabled and 0.8 or 0.2, Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.new(1, -4, 1, -4), Parent = m})
            O = true
        end
    end
    if not O then
        m.Text = j.Text
    end
    j.Base = m
    j.Stroke = K
    j.UpdateColors = function(Y)
        if f.Unloaded then return
    end
    _(j.Tween)
    m.BackgroundColor3 = j.Disabled and f.Scheme.BackgroundColor or f.Scheme.MainColor
    m.TextTransparency = j.Disabled and 0.8 or 0.35
    K.Transparency = j.Disabled and 0.5 or 0
    m.Active = not j.Disabled
    f.Registry[m].BackgroundColor3 = j.Disabled and "BackgroundColor" or "MainColor"
end
j.SetDisabled = function(Y, Y)
    j.Disabled = Y
    if j.TooltipTable then
        j.TooltipTable.Disabled = j.Disabled
    end
    j:UpdateColors()
end
j.SetVisible = function(Y, Y)
    j.Visible = Y
    m.Visible = Y
end
j.SetText = function(Y, Y)
    j.Text = Y
    m.Text = Y
end
m.MouseEnter:Connect(function()
    if j.Disabled then return
end
f:PlaySound("hover", 0.06, 1.5)
j.Tween = B:Create(m, f.TweenInfo, {TextTransparency = 0})
j.Tween:Play()
end)
m.MouseLeave:Connect(function()
    if j.Disabled then return
end
j.Tween = B:Create(m, f.TweenInfo, {TextTransparency = 0.35})
j.Tween:Play()
end)
m.MouseButton1Click:Connect(function()
    if j.Disabled then return
end
f:PlaySound("click", 0.12, 1)
f:SafeCallback(j.Func)
end)
if typeof(j.Tooltip) == "string" or typeof(j.DisabledTooltip) == "string" then
    j.TooltipTable = f:AddTooltip(j.Tooltip, j.DisabledTooltip, m)
    j.TooltipTable.Disabled = j.Disabled
end
if j.Risky then
    m.TextColor3 = f.Scheme.RedColor
    f.Registry[m].TextColor3 = "RedColor"
end
j:UpdateColors()
if s.Addons then
    table.insert(s.Addons, j)
end
return j
end
I.__index = x
I.__namecall = function(s, s, ...)
    return x[s](...)
end
end
local s = {}
do
    local Y = {}
    local function x(K)
        f.Registry[K] = nil
        for j, j in ipairs(K:GetDescendants()) do
            f.Registry[j] = nil
        end
    end
    Y.RemoveElement = function(K, j)
        if not j then return
    end
    pcall(function()
        if j.Menu and j.Menu.Destroy then j.Menu:Destroy()
    end
end)
pcall(function()
    if j.TooltipTable and j.TooltipTable.Destroy then j.TooltipTable:Destroy()
end
end)
local O = {H, z, J}
for m, m in ipairs(O) do
    for O, k in pairs(m) do
        if k == j then
            m[O] = nil
        end
    end
end
local O = j.Holder or j.Base
if O then
    pcall(function()
        x(O)
    end)
    pcall(function()
        O:Destroy()
    end)
end
for x = #K.Elements, 1, -1 do
    if K.Elements[x] == j then
        table.remove(K.Elements, x)
    end
end
pcall(function()
    K:Resize()
end)
end
Y.Clear = function(x)
    for K = #x.Elements, 1, -1 do
        x:RemoveElement(x.Elements[K])
    end
end
Y.AddDivider = function(x, K)
    local j = x.Container
    local O = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 6), Parent = j})
    if K then
        j = F("TextLabel", {AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), Text = K, TextSize = 14, TextTransparency = 0.5, TextXAlignment = Enum.TextXAlignment.Center, Parent = O})
        local m = f:GetTextBounds(K, j.FontFace, j.TextSize, j.AbsoluteSize.X)
        local K = math.floor(m / 2) + 10
        F("Frame", {AnchorPoint = Vector2.new(0, 0.5), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 0.5), Size = UDim2.new(0.5, -K, 0, 2), Parent = O})
        F("Frame", {AnchorPoint = Vector2.new(1, 0.5), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(1, 0.5), Size = UDim2.new(0.5, -K, 0, 2), Parent = O})
    else
        F("Frame", {AnchorPoint = Vector2.new(0, 0.5), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 0.5), Size = UDim2.new(1, 0, 0, 2), Parent = O})
    end
    x:Resize()
    table.insert(x.Elements, {Holder = O, Type = "Divider"})
end
Y.AddLabel = function(x, ...)
    local K, j, O = {}, select(1, ...), select(2, ...)
    if typeof(j) == "table" or typeof(O) == "table" then
        local m = typeof(j) == "table" and j or O
        K.Text = m.Text or ""
        K.DoesWrap = m.DoesWrap or false
        K.Size = m.Size or 14
        K.Visible = m.Visible or true
        K.Idx = typeof(O) == "table" and j or nil
    else
        K.Text = j or ""
        K.DoesWrap = O or false
        K.Size = 14
        K.Visible = true
        K.Idx = select(3, ...) or nil
    end
    local j = x
    local m, k = j.Container, {Text = K.Text, DoesWrap = K.DoesWrap, Addons = {}, Visible = K.Visible, Type = "Label"}
    local n = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 18), Text = k.Text, TextSize = K.Size, TextWrapped = k.DoesWrap, TextXAlignment = j.IsKeyTab and Enum.TextXAlignment.Center or Enum.TextXAlignment.Left, Parent = m})
    k.SetVisible = function(E, E)
        k.Visible = E
        n.Visible = k.Visible
        j:Resize()
    end
    k.SetText = function(E, S)
        k.Text = S
        n.Text = S
        if k.DoesWrap then
            E, S = f:GetTextBounds(k.Text, n.FontFace, n.TextSize, n.AbsoluteSize.X)
            n.Size = UDim2.new(1, 0, 0, S + 4)
        end
        j:Resize()
    end
    if k.DoesWrap then
        O, x = f:GetTextBounds(k.Text, n.FontFace, n.TextSize, n.AbsoluteSize.X)
        n.Size = UDim2.new(1, 0, 0, x + 4)
        local x = n.AbsoluteSize
        n:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
            if n.AbsoluteSize == x then return
        end
        local O, O = f:GetTextBounds(k.Text, n.FontFace, n.TextSize, n.AbsoluteSize.X)
        n.Size = UDim2.new(1, 0, 0, O + 4)
        x = n.AbsoluteSize
        j:Resize()
    end)
else
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Right, Padding = UDim.new(0, 6), Parent = n})
end
j:Resize()
k.TextLabel = n
k.Container = m
if not K.DoesWrap then
    setmetatable(k, I)
end
k.Holder = n
table.insert(j.Elements, k)
if K.Idx then
    J[K.Idx] = k
else
    table.insert(J, k)
end
return k
end
Y.AddButton = function(J, ...)
    local function x(...)
        local K, j, O = {}, select(1, ...), select(2, ...)
        if typeof(j) == "table" or typeof(O) == "table" then
            local m = typeof(j) == "table" and j or O
            K.Text = m.Text or ""
            K.Func = m.Func or m.Callback or function()
            end
            K.DoubleClick = m.DoubleClick
            K.Tooltip = m.Tooltip
            K.DisabledTooltip = m.DisabledTooltip
            K.Risky = m.Risky or false
            K.Disabled = m.Disabled or false
            K.Visible = m.Visible or true
            K.Idx = typeof(O) == "table" and j or nil
        else
            K.Text = j or ""
            K.Func = O or function()
            end
            K.DoubleClick = false
            K.Tooltip = nil
            K.DisabledTooltip = nil
            K.Risky = false
            K.Disabled = false
            K.Visible = true
            K.Idx = select(3, ...) or nil
        end
        return K
    end
    local K, j = x(...), J
    local J, O = j.Container, {Text = K.Text, Func = K.Func, DoubleClick = K.DoubleClick, Tooltip = K.Tooltip, DisabledTooltip = K.DisabledTooltip, TooltipTable = nil, Risky = K.Risky, Disabled = K.Disabled, Visible = K.Visible, Tween = nil, Type = "Button"}
    local m = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 21), Parent = J})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalFlex = Enum.UIFlexAlignment.Fill, Padding = UDim.new(0, 9), Parent = m})
    local function J(k)
        local n = F("TextButton", {Active = not k.Disabled, BackgroundColor3 = k.Disabled and "BackgroundColor" or "MainColor", Size = UDim2.fromScale(1, 1), Text = k.Text, TextSize = 14, TextTransparency = 0.3, Visible = k.Visible, Parent = m})
        return n, (F("UIStroke", {Color = "OutlineColor", Transparency = k.Disabled and 0.5 or 0, Parent = n}))
    end
    local function k(n)
        n.Base.MouseEnter:Connect(function()
            if n.Disabled then return
        end
        f:PlaySound("hover", 0.08, 1.5)
        n.Tween = B:Create(n.Base, f.TweenInfo, {TextTransparency = 0})
        n.Tween:Play()
    end)
    n.Base.MouseLeave:Connect(function()
        if n.Disabled then return
    end
    n.Tween = B:Create(n.Base, f.TweenInfo, {TextTransparency = 0.4})
    n.Tween:Play()
end)
n.Base.MouseButton1Click:Connect(function()
    if n.Disabled or n.Locked then return
end
f:PlaySound("click", 0.15, 1)
if n.DoubleClick then
    n.Locked = true
    n.Base.Text = "Are you sure?"
    n.Base.TextColor3 = f.Scheme.AccentColor
    f.Registry[n.Base].TextColor3 = "AccentColor"
    local E = Z(n.Base.MouseButton1Click, 0.5)
    n.Base.Text = n.Text
    n.Base.TextColor3 = n.Risky and f.Scheme.RedColor or f.Scheme.FontColor
    f.Registry[n.Base].TextColor3 = n.Risky and "RedColor" or "FontColor"
    if E then
        f:SafeCallback(n.Func)
    end
    q.RenderStepped:Wait()
    n.Locked = false
    return
end
f:SafeCallback(n.Func)
end)
end
O.Base, O.Stroke = J(O)
k(O)
O.AddButton = function(Z, ...)
    Z = x(...)
    local x = {Text = Z.Text, Func = Z.Func, DoubleClick = Z.DoubleClick, Tooltip = Z.Tooltip, DisabledTooltip = Z.DisabledTooltip, TooltipTable = nil, Risky = Z.Risky, Disabled = Z.Disabled, Visible = Z.Visible, Tween = nil, Type = "SubButton"}
    O.SubButton = x
    x.Base, x.Stroke = J(x)
    k(x)
    x.UpdateColors = function(J)
        if f.Unloaded then return
    end
    _(x.Tween)
    x.Base.BackgroundColor3 = x.Disabled and f.Scheme.BackgroundColor or f.Scheme.MainColor
    x.Base.TextTransparency = x.Disabled and 0.8 or 0.4
    x.Stroke.Transparency = x.Disabled and 0.5 or 0
    f.Registry[x.Base].BackgroundColor3 = x.Disabled and "BackgroundColor" or "MainColor"
end
x.SetDisabled = function(J, J)
    x.Disabled = J
    if x.TooltipTable then
        x.TooltipTable.Disabled = x.Disabled
    end
    x.Base.Active = not x.Disabled
    x:UpdateColors()
end
x.SetVisible = function(J, J)
    x.Visible = J
    x.Base.Visible = x.Visible
    j:Resize()
end
x.SetText = function(J, J)
    x.Text = J
    x.Base.Text = J
end
if typeof(x.Tooltip) == "string" or typeof(x.DisabledTooltip) == "string" then
    x.TooltipTable = f:AddTooltip(x.Tooltip, x.DisabledTooltip, x.Base)
    x.TooltipTable.Disabled = x.Disabled
end
if x.Risky then
    x.Base.TextColor3 = f.Scheme.RedColor
    f.Registry[x.Base].TextColor3 = "RedColor"
end
x:UpdateColors()
if Z.Idx then
    o[Z.Idx] = x
else
    table.insert(o, x)
end
return x
end
O.UpdateColors = function(J)
    if f.Unloaded then return
end
_(O.Tween)
O.Base.BackgroundColor3 = O.Disabled and f.Scheme.BackgroundColor or f.Scheme.MainColor
O.Base.TextTransparency = O.Disabled and 0.8 or 0.4
O.Stroke.Transparency = O.Disabled and 0.5 or 0
f.Registry[O.Base].BackgroundColor3 = O.Disabled and "BackgroundColor" or "MainColor"
end
O.SetDisabled = function(J, J)
    O.Disabled = J
    if O.TooltipTable then
        O.TooltipTable.Disabled = O.Disabled
    end
    O.Base.Active = not O.Disabled
    O:UpdateColors()
end
O.SetVisible = function(J, J)
    O.Visible = J
    m.Visible = O.Visible
    j:Resize()
end
O.SetText = function(J, J)
    O.Text = J
    O.Base.Text = J
end
if typeof(O.Tooltip) == "string" or typeof(O.DisabledTooltip) == "string" then
    O.TooltipTable = f:AddTooltip(O.Tooltip, O.DisabledTooltip, O.Base)
    O.TooltipTable.Disabled = O.Disabled
end
if O.Risky then
    O.Base.TextColor3 = f.Scheme.RedColor
    f.Registry[O.Base].TextColor3 = "RedColor"
end
O:UpdateColors()
j:Resize()
O.Holder = m
table.insert(j.Elements, O)
if K.Idx then
    o[K.Idx] = O
else
    table.insert(o, O)
end
return O
end
Y.AddCheckbox = function(J, o, Z)
    Z = f:Validate(Z, l.Toggle)
    local _ = J
    local J, x = _.Container, {Text = Z.Text, Value = Z.Default, Tooltip = Z.Tooltip, DisabledTooltip = Z.DisabledTooltip, TooltipTable = nil, Callback = Z.Callback, Changed = Z.Changed, Risky = Z.Risky, Disabled = Z.Disabled, Visible = Z.Visible, Addons = {}, Type = "Toggle"}
    local Z = F("TextButton", {Active = not x.Disabled, BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 18), Text = "", Visible = x.Visible, Parent = J})
    local K = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(26, 0), Size = UDim2.new(1, -26, 1, 0), Text = x.Text, TextSize = 14, TextTransparency = 0.4, TextXAlignment = Enum.TextXAlignment.Left, Parent = Z})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Right, Padding = UDim.new(0, 6), Parent = K})
    local j = F("Frame", {BackgroundColor3 = "MainColor", Size = UDim2.fromScale(1, 1), SizeConstraint = Enum.SizeConstraint.RelativeYY, Parent = Z})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius / 2), Parent = j})
    local O, m = F("UIStroke", {Color = "OutlineColor", Parent = j}), F("ImageLabel", {Image = L and L.Url or "", ImageColor3 = "FontColor", ImageRectOffset = L and L.ImageRectOffset or Vector2.zero, ImageRectSize = L and L.ImageRectSize or Vector2.zero, ImageTransparency = 1, Position = UDim2.fromOffset(2, 2), Size = UDim2.new(1, -4, 1, -4), Parent = j})
    x.UpdateColors = function(k)
        x:Display()
    end
    x.Display = function(k)
        if f.Unloaded then return
    end
    local function k()
        O.Transparency = x.Disabled and 0.5 or 0
        if x.Disabled then
            K.TextTransparency = 0.8
            m.ImageTransparency = x.Value and 0.8 or 1
            j.BackgroundColor3 = f.Scheme.BackgroundColor
            f.Registry[j].BackgroundColor3 = "BackgroundColor"
            return
        end
        B:Create(K, f.TweenInfo, {TextTransparency = x.Value and 0 or 0.4}):Play()
        B:Create(m, f.TweenInfo, {ImageTransparency = x.Value and 0 or 1}):Play()
        j.BackgroundColor3 = f.Scheme.MainColor
        f.Registry[j].BackgroundColor3 = "MainColor"
    end
    if yabujinCoreState and yabujinCoreState._withId then
        yabujinCoreState._withId(k)
    else
        pcall(k)
    end
end
x.OnChanged = function(j, j)
    x.Changed = j
end
x.SetValue = function(j, j)
    if x.Disabled then return
end
x.Value = j
x:Display()
for j, j in pairs(x.Addons) do
    if j.Type == "KeyPicker" and j.SyncToggleState then
        j.Toggled = x.Value
        j:Update()
    end
end
f:UpdateDependencyBoxes()
f:SafeCallback(x.Callback, x.Value)
f:SafeCallback(x.Changed, x.Value)
end
x.SetDisabled = function(j, j)
    x.Disabled = j
    if x.TooltipTable then
        x.TooltipTable.Disabled = x.Disabled
    end
    for j, j in pairs(x.Addons) do
        if j.Type == "KeyPicker" and j.SyncToggleState then
            j:Update()
        end
    end
    Z.Active = not x.Disabled
    x:Display()
end
x.SetVisible = function(j, j)
    x.Visible = j
    Z.Visible = x.Visible
    _:Resize()
end
x.SetText = function(j, j)
    x.Text = j
    K.Text = j
end
Z.MouseButton1Click:Connect(function()
    if x.Disabled then return
end
local j = not x.Value
f:PlaySound(j and "toggle_on" or "toggle_off", 0.12, j and 1.1 or 0.9)
x:SetValue(j)
end)
if typeof(x.Tooltip) == "string" or typeof(x.DisabledTooltip) == "string" then
    x.TooltipTable = f:AddTooltip(x.Tooltip, x.DisabledTooltip, Z)
    x.TooltipTable.Disabled = x.Disabled
end
if x.Risky then
    K.TextColor3 = f.Scheme.RedColor
    f.Registry[K].TextColor3 = "RedColor"
end
x:Display()
_:Resize()
x.TextLabel = K
x.Container = J
setmetatable(x, I)
x.Holder = Z
table.insert(_.Elements, x)
x.Default = x.Value
H[o] = x
return x
end
Y.AddToggle = function(J, o, Z)
    Z = f:Validate(Z, l.Toggle)
    local _ = J
    local J, x = _.Container, {Text = Z.Text, Value = Z.Default, Tooltip = Z.Tooltip, DisabledTooltip = Z.DisabledTooltip, TooltipTable = nil, Callback = Z.Callback, Changed = Z.Changed, Risky = Z.Risky, Disabled = Z.Disabled, Visible = Z.Visible, Addons = {}, Type = "Toggle"}
    local Z = F("TextButton", {Active = not x.Disabled, BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 18), Text = "", Visible = x.Visible, Parent = J})
    local K = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(26, 0), Size = UDim2.new(1, -26, 1, 0), Text = x.Text, TextSize = 14, TextTransparency = 0.4, TextXAlignment = Enum.TextXAlignment.Left, Parent = Z})
    F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Right, Padding = UDim.new(0, 6), Parent = K})
    local j = F("Frame", {BackgroundColor3 = "MainColor", Size = UDim2.fromScale(1, 1), SizeConstraint = Enum.SizeConstraint.RelativeYY, Parent = Z})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius / 2), Parent = j})
    local O, m = F("UIStroke", {Color = "OutlineColor", Parent = j}), F("ImageLabel", {Image = L and L.Url or "", ImageColor3 = "FontColor", ImageRectOffset = L and L.ImageRectOffset or Vector2.zero, ImageRectSize = L and L.ImageRectSize or Vector2.zero, ImageTransparency = 1, Position = UDim2.fromOffset(2, 2), Size = UDim2.new(1, -4, 1, -4), Parent = j})
    x.UpdateColors = function(L)
        x:Display()
    end
    x.Display = function(L)
        if f.Unloaded then return
    end
    local function L()
        O.Transparency = x.Disabled and 0.5 or 0
        if x.Disabled then
            K.TextTransparency = 0.8
            m.ImageTransparency = x.Value and 0.8 or 1
            j.BackgroundColor3 = f.Scheme.BackgroundColor
            f.Registry[j].BackgroundColor3 = "BackgroundColor"
            return
        end
        B:Create(K, f.TweenInfo, {TextTransparency = x.Value and 0 or 0.4}):Play()
        B:Create(m, f.TweenInfo, {ImageTransparency = x.Value and 0 or 1}):Play()
        j.BackgroundColor3 = f.Scheme.MainColor
        f.Registry[j].BackgroundColor3 = "MainColor"
    end
    if yabujinCoreState and yabujinCoreState._withId then
        yabujinCoreState._withId(L)
    else
        pcall(L)
    end
end
x.OnChanged = function(L, L)
    x.Changed = L
end
x.SetValue = function(L, L)
    if x.Disabled then return
end
x.Value = L
x:Display()
for L, L in pairs(x.Addons) do
    if L.Type == "KeyPicker" and L.SyncToggleState then
        L.Toggled = x.Value
        L:Update()
    end
end
f:UpdateDependencyBoxes()
f:SafeCallback(x.Callback, x.Value)
f:SafeCallback(x.Changed, x.Value)
end
x.SetDisabled = function(L, L)
    x.Disabled = L
    if x.TooltipTable then
        x.TooltipTable.Disabled = x.Disabled
    end
    for L, L in pairs(x.Addons) do
        if L.Type == "KeyPicker" and L.SyncToggleState then
            L:Update()
        end
    end
    Z.Active = not x.Disabled
    x:Display()
end
x.SetVisible = function(L, L)
    x.Visible = L
    Z.Visible = x.Visible
    _:Resize()
end
x.SetText = function(L, L)
    x.Text = L
    K.Text = L
end
Z.MouseButton1Click:Connect(function()
    if x.Disabled then return
end
local L = not x.Value
f:PlaySound(L and "toggle_on" or "toggle_off", 0.12, L and 1.1 or 0.9)
x:SetValue(L)
end)
if typeof(x.Tooltip) == "string" or typeof(x.DisabledTooltip) == "string" then
    x.TooltipTable = f:AddTooltip(x.Tooltip, x.DisabledTooltip, Z)
    x.TooltipTable.Disabled = x.Disabled
end
if x.Risky then
    K.TextColor3 = f.Scheme.RedColor
    f.Registry[K].TextColor3 = "RedColor"
end
x:Display()
_:Resize()
x.TextLabel = K
x.Container = J
setmetatable(x, I)
x.Holder = Z
table.insert(_.Elements, x)
x.Default = x.Value
H[o] = x
return x
end
Y.AddInput = function(J, L, o)
    o = f:Validate(o, l.Input)
    local H = J
    local J, Z = H.Container, {Text = o.Text, Value = o.Default, Finished = o.Finished, Numeric = o.Numeric, ClearTextOnFocus = o.ClearTextOnFocus, Placeholder = o.Placeholder, AllowEmpty = o.AllowEmpty, EmptyReset = o.EmptyReset, Tooltip = o.Tooltip, DisabledTooltip = o.DisabledTooltip, TooltipTable = nil, Callback = o.Callback, Changed = o.Changed, Disabled = o.Disabled, Visible = o.Visible, Type = "Input"}
    local _ = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 39), Visible = Z.Visible, Parent = J})
    local J, I = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 14), Text = Z.Text, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = _}), F("TextBox", {AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, ClearTextOnFocus = not Z.Disabled and Z.ClearTextOnFocus, PlaceholderText = Z.Placeholder, Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 21), Text = Z.Value, TextEditable = not Z.Disabled, TextScaled = true, TextXAlignment = Enum.TextXAlignment.Left, Parent = _})
    F("UIPadding", {PaddingBottom = UDim.new(0, 3), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 4), Parent = I})
    Z.UpdateColors = function(x)
        if f.Unloaded then return
    end
    J.TextTransparency = Z.Disabled and 0.8 or 0
    I.TextTransparency = Z.Disabled and 0.8 or 0
end
Z.OnChanged = function(x, x)
    Z.Changed = x
end
Z.SetValue = function(x, x)
    if x == nil then x = Z.EmptyReset or(Z.Numeric and "0" or "")
end
if not Z.AllowEmpty and c(x) == "" then
    x = Z.EmptyReset or(Z.Numeric and "0" or "")
end
if o.MaxLength and#x > o.MaxLength then
    x = x:sub(1, o.MaxLength)
end
if Z.Numeric then
    if#tostring(x) > 0 and not tonumber(x) then
        x = Z.Value or "0"
    end
end
Z.Value = x
local function o()
    I.Text = tostring(x)
end
if yabujinCoreState and yabujinCoreState._withId then
    yabujinCoreState._withId(o)
else
    pcall(o)
end
if not Z.Disabled then
    f:SafeCallback(Z.Callback, Z.Value)
    f:SafeCallback(Z.Changed, Z.Value)
end
end
Z.SetDisabled = function(o, o)
    Z.Disabled = o
    if Z.TooltipTable then
        Z.TooltipTable.Disabled = Z.Disabled
    end
    I.ClearTextOnFocus = not Z.Disabled and Z.ClearTextOnFocus
    I.TextEditable = not Z.Disabled
    Z:UpdateColors()
end
Z.SetVisible = function(o, o)
    Z.Visible = o
    _.Visible = Z.Visible
    H:Resize()
end
Z.SetText = function(o, o)
    Z.Text = o
    J.Text = o
end
if Z.Finished then
    I.FocusLost:Connect(function(J)
        if not J then return
    end
    Z:SetValue(I.Text)
end)
else
    I:GetPropertyChangedSignal("Text"):Connect(function()
        Z:SetValue(I.Text)
    end)
end
if typeof(Z.Tooltip) == "string" or typeof(Z.DisabledTooltip) == "string" then
    Z.TooltipTable = f:AddTooltip(Z.Tooltip, Z.DisabledTooltip, I)
    Z.TooltipTable.Disabled = Z.Disabled
end
H:Resize()
Z.Holder = _
table.insert(H.Elements, Z)
Z.Default = Z.Value
z[L] = Z
return Z
end
Y.AddSlider = function(J, L, o)
    o = f:Validate(o, l.Slider)
    local H = J
    local J, Z = H.Container, {Text = o.Text, Value = o.Default, Min = o.Min, Max = o.Max, Prefix = o.Prefix, Suffix = o.Suffix, Compact = o.Compact, Rounding = o.Rounding, Tooltip = o.Tooltip, DisabledTooltip = o.DisabledTooltip, TooltipTable = nil, Callback = o.Callback, Changed = o.Changed, Disabled = o.Disabled, Visible = o.Visible, Type = "Slider"}
    local _ = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, o.Compact and 13 or 31), Visible = Z.Visible, Parent = J})
    local J
    if not o.Compact then
        J = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 14), Text = Z.Text, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = _})
    end
    local c = F("TextButton", {Active = not Z.Disabled, AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 8), Text = "", Parent = _})
    F("UICorner", {CornerRadius = UDim.new(0, 4), Parent = c})
    local I = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = "", TextSize = 14, ZIndex = 2, Parent = c})
    F("UIStroke", {ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual, Color = "DarkColor", LineJoinMode = Enum.LineJoinMode.Miter, Parent = I})
    local x = F("Frame", {BackgroundColor3 = "AccentColor", Size = UDim2.fromScale(0.5, 1), Parent = c})
    F("UICorner", {CornerRadius = UDim.new(0, 4), Parent = x})
    Z.UpdateColors = function(K)
        if f.Unloaded then return
    end
    if J then
        J.TextTransparency = Z.Disabled and 0.8 or 0
    end
    I.TextTransparency = Z.Disabled and 0.8 or 0
    x.BackgroundColor3 = Z.Disabled and f.Scheme.OutlineColor or f.Scheme.AccentColor
    f.Registry[x].BackgroundColor3 = Z.Disabled and "OutlineColor" or "AccentColor"
end
Z.Display = function(K)
    if f.Unloaded then return
end
K = if o.FormatDisplayValue then(o.FormatDisplayValue(Z, Z.Value)) else nil
if K then
    I.Text = tostring(K)
elseif o.Compact then
    I.Text = string.format("%s: %s%s%s", Z.Text, Z.Prefix, Z.Value, Z.Suffix)
elseif o.HideMax then
    I.Text = string.format("%s%s%s", Z.Prefix, Z.Value, Z.Suffix)
else
    I.Text = string.format("%s%s%s/%s%s%s", Z.Prefix, Z.Value, Z.Suffix, Z.Prefix, Z.Max, Z.Suffix)
end
K = (Z.Value -Z.Min) / (Z.Max -Z.Min)
x.Size = UDim2.fromScale(K, 1)
end
Z.OnChanged = function(o, o)
    Z.Changed = o
end
Z.SetMax = function(o, o)
    assert(o > Z.Min, "Max value cannot be less than the current min value.")
    Z:SetValue(math.clamp(Z.Value, Z.Min, o))
    Z.Max = o
    Z:Display()
end
Z.SetMin = function(o, o)
    assert(o < Z.Max, "Min value cannot be greater than the current max value.")
    Z:SetValue(math.clamp(Z.Value, o, Z.Max))
    Z.Min = o
    Z:Display()
end
Z.SetValue = function(o, I)
    if Z.Disabled then return
end
o = tonumber(I)
if not o or o == Z.Value then
    return
end
Z.Value = math.clamp(o, Z.Min, Z.Max)
Z:Display()
f:SafeCallback(Z.Callback, Z.Value)
f:SafeCallback(Z.Changed, Z.Value)
end
Z.SetDisabled = function(o, o)
    Z.Disabled = o
    if Z.TooltipTable then
        Z.TooltipTable.Disabled = Z.Disabled
    end
    c.Active = not Z.Disabled
    Z:UpdateColors()
end
Z.SetVisible = function(o, o)
    Z.Visible = o
    _.Visible = Z.Visible
    H:Resize()
end
Z.SetText = function(o, o)
    Z.Text = o
    if J then
        J.Text = o
        return
    end
    Z:Display()
end
Z.SetPrefix = function(J, J)
    Z.Prefix = J
    Z:Display()
end
Z.SetSuffix = function(J, J)
    Z.Suffix = J
    Z:Display()
end
c.InputBegan:Connect(function(J)
    if not a(J) or Z.Disabled then return
end
for o, o in pairs(f.ActiveTab.Sides) do
    o.ScrollingEnabled = false
end
while g(J) do
    local J = C.X
    local o, g = math.clamp((J -c.AbsolutePosition.X) / c.AbsoluteSize.X, 0, 1), Z.Value
    Z.Value = e(Z.Min + (Z.Max -Z.Min) * o, Z.Rounding)
    Z:Display()
    if Z.Value ~= g then
        f:SafeCallback(Z.Callback, Z.Value)
        f:SafeCallback(Z.Changed, Z.Value)
    end
    q.RenderStepped:Wait()
end
for J, J in pairs(f.ActiveTab.Sides) do
    J.ScrollingEnabled = true
end
end)
if typeof(Z.Tooltip) == "string" or typeof(Z.DisabledTooltip) == "string" then
    Z.TooltipTable = f:AddTooltip(Z.Tooltip, Z.DisabledTooltip, c)
    Z.TooltipTable.Disabled = Z.Disabled
end
Z:UpdateColors()
Z:Display()
H:Resize()
Z.Holder = _
table.insert(H.Elements, Z)
Z.Default = Z.Value
z[L] = Z
return Z
end
Y.AddDropdown = function(J, L, o)
    o = f:Validate(o, l.Dropdown)
    local H = J
    J = H.Container
    if o.SpecialType == "Player" then
        o.Values = r(o.ExcludeLocalPlayer)
        o.AllowNull = true
    elseif o.SpecialType == "Team" then
        o.Values = u()
        o.AllowNull = true
    end
    local Z = {Text = typeof(o.Text) == "string" and o.Text or nil, Value = o.Multi and{} or nil, Values = o.Values, DisabledValues = o.DisabledValues, Multi = o.Multi, SpecialType = o.SpecialType, ExcludeLocalPlayer = o.ExcludeLocalPlayer, Tooltip = o.Tooltip, DisabledTooltip = o.DisabledTooltip, TooltipTable = nil, Callback = o.Callback, Changed = o.Changed, Disabled = o.Disabled, Visible = o.Visible, Type = "Dropdown"}
    local g = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, Z.Text and 39 or 21), Visible = Z.Visible, Parent = J})
    local _, c = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 14), Text = Z.Text, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Visible = not not o.Text, Parent = g}), F("TextButton", {Active = not Z.Disabled, AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 21), Text = "---", TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = g})
    F("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 4), Parent = c})
    local e = F("ImageLabel", {AnchorPoint = Vector2.new(1, 0.5), Image = P and P.Url or "", ImageColor3 = "FontColor", ImageRectOffset = P and P.ImageRectOffset or Vector2.zero, ImageRectSize = P and P.ImageRectSize or Vector2.zero, ImageTransparency = 0.5, Position = UDim2.fromScale(1, 0.5), Size = UDim2.fromOffset(16, 16), Parent = c})
    local P
    if o.Searchable then
        P = F("TextBox", {BackgroundTransparency = 1, PlaceholderText = "Search...", Position = UDim2.fromOffset(- 8, 0), Size = UDim2.new(1, -12, 1, 0), TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Visible = false, Parent = c})
        F("UIPadding", {PaddingLeft = UDim.new(0, 8), Parent = P})
    end
    local I = f:AddContextMenu(c, function()
        return UDim2.fromOffset(c.AbsoluteSize.X / f.DPIScale, 0)
    end, function()
        return{0.5, c.AbsoluteSize.Y + 1.5}
    end, 2, function(x)
        c.TextTransparency = x and P and 1 or 0
        e.ImageTransparency = x and 0 or 0.5
        e.Rotation = x and 180 or 0
        if P then
            P.Text = ""
            P.Visible = x
        end
    end)
    Z.Menu = I
    Z.RecalculateListSize = function(x, x)
        local K = math.clamp((x or(p(Z.Values))) * 21, 0, o.MaxVisibleDropdownItems * 21)
        I:SetSize(function()
            return UDim2.fromOffset(c.AbsoluteSize.X / f.DPIScale, K)
        end)
    end
    Z.UpdateColors = function(p)
        if f.Unloaded then return
    end
    _.TextTransparency = Z.Disabled and 0.8 or 0
    c.TextTransparency = Z.Disabled and 0.8 or 0
    e.ImageTransparency = Z.Disabled and 0.8 or I.Active and 0 or 0.5
end
Z.Display = function(p)
    if f.Unloaded then return
end
p = ""
if o.Multi then
    for e, e in pairs(Z.Values) do
        p = if Z.Value[e] then p .. (o.FormatDisplayValue and(tostring(o.FormatDisplayValue(e))) or(tostring(e))) .. ", " else p
    end
    p = (p:sub(1, #p -2))
else
    p = Z.Value and(tostring(Z.Value)) or ""
    p = if p ~= "" and o.FormatDisplayValue then(tostring(o.FormatDisplayValue(p))) else p
end
c.Text = (if#p > 25 then p:sub(1, 22) .. "..." else p) == "" and "---" or(if#p > 25 then p:sub(1, 22) .. "..." else p)
end
Z.OnChanged = function(p, p)
    Z.Changed = p
end
Z.GetActiveValues = function(p)
    if o.Multi then p = {}
    for e, x in pairs(Z.Value) do
        table.insert(p, e)
    end
    return p
end
return Z.Value and 1 or 0
end
local p = {}
Z.BuildDropdownList = function(e)
    local x, K = Z.Values, Z.DisabledValues
    for j, O in pairs(p) do
        j:Destroy()
    end
    table.clear(p)
    e = 0
    for j, j in pairs(x) do
        if not P or(tostring(j):lower():match(P.Text:lower())) then
            e += 1
            local x, O = table.find(K, j), {}
            local K = F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 1, LayoutOrder = x and 1 or 0, Size = UDim2.new(1, 0, 0, 21), Text = tostring(j), TextSize = 14, TextTransparency = 0.5, TextXAlignment = Enum.TextXAlignment.Left, Parent = I.Menu})
            F("UIPadding", {PaddingLeft = UDim.new(0, 7), PaddingRight = UDim.new(0, 7), Parent = K})
            local m
            if o.Multi then
                m = Z.Value[j]
            else
                m = Z.Value == j
            end
            O.UpdateButton = function(k)
                m = if o.Multi then Z.Value[j] else Z.Value == j
                K.BackgroundTransparency = m and 0 or 1
                K.TextTransparency = x and 0.8 or m and 0 or 0.5
            end
            if not x then
                K.MouseButton1Click:Connect(function()
                    local x = not m
                    if not(Z:GetActiveValues() == 1 and not x and not o.AllowNull) then
                        m = x
                        if o.Multi then
                            Z.Value[j] = m and true or nil
                        else
                            Z.Value = m and j or nil
                        end
                        for x, x in pairs(p) do
                            x:UpdateButton()
                        end
                    end
                    O:UpdateButton()
                    Z:Display()
                    f:UpdateDependencyBoxes()
                    f:SafeCallback(Z.Callback, Z.Value)
                    f:SafeCallback(Z.Changed, Z.Value)
                end)
            end
            O:UpdateButton()
            Z:Display()
            p[K] = O
        end
    end
    Z:RecalculateListSize(e)
end
Z.SetValue = function(e, x)
    if o.Multi then e = {}
    for K, j in x or{}, nil, nil do
        if typeof(j) ~= "boolean" then
            e[j] = true
        elseif j and(table.find(Z.Values, K)) then
            e[K] = true
        end
    end
    Z.Value = e
elseif table.find(Z.Values, x) then
    Z.Value = x
elseif not x then
    Z.Value = nil
end
Z:Display()
for e, e in pairs(p) do
    e:UpdateButton()
end
if not Z.Disabled then
    f:UpdateDependencyBoxes()
    f:SafeCallback(Z.Callback, Z.Value)
    f:SafeCallback(Z.Changed, Z.Value)
end
end
Z.SetValues = function(p, p)
    Z.Values = p
    Z:BuildDropdownList()
end
Z.SetMulti = function(p, e)
    e = not not e
    if o.Multi == e then
        return
    end
    o.Multi = e
    Z.Multi = e
    if e then
        local e, x = Z.Value, {}
        if typeof(e) == "string" then
            x[e] = true
        end
        Z.Value = x
    else
        p = nil
        if typeof(Z.Value) == "table" then
            for e, e in ipairs(Z.Values) do
                if Z.Value[e] then
                    p = e
                    break
                end
            end
        end
        Z.Value = p
    end
    Z:BuildDropdownList()
    Z:Display()
end
Z.AddValues = function(p, p)
    if typeof(p) == "table" then for e, e in pairs(p) do
        table.insert(Z.Values, e)
    end
elseif typeof(p) == "string" then
    table.insert(Z.Values, p)
else
    return
end
Z:BuildDropdownList()
end
Z.SetDisabledValues = function(p, p)
    Z.DisabledValues = p
    Z:BuildDropdownList()
end
Z.AddDisabledValues = function(p, p)
    if typeof(p) == "table" then for e, e in pairs(p) do
        table.insert(Z.DisabledValues, e)
    end
elseif typeof(p) == "string" then
    table.insert(Z.DisabledValues, p)
else
    return
end
Z:BuildDropdownList()
end
Z.SetDisabled = function(p, p)
    Z.Disabled = p
    if Z.TooltipTable then
        Z.TooltipTable.Disabled = Z.Disabled
    end
    I:Close()
    c.Active = not Z.Disabled
    Z:UpdateColors()
end
Z.SetVisible = function(p, p)
    Z.Visible = p
    g.Visible = Z.Visible
    H:Resize()
end
Z.SetText = function(p, p)
    Z.Text = p
    g.Size = UDim2.new(1, 0, 0, p and 39 or 21)
    _.Text = p or ""
    _.Visible = not not p
end
c.MouseButton1Click:Connect(function()
    if Z.Disabled then return
end
I:Toggle()
end)
if P then
    P:GetPropertyChangedSignal("Text"):Connect(Z.BuildDropdownList)
end
J = {}
if typeof(o.Default) == "string" then
    local P = table.find(Z.Values, o.Default)
    if P then
        table.insert(J, P)
    end
elseif typeof(o.Default) == "table" then
    for P, p in next, o.Default, nil do
        P = table.find(Z.Values, p)
        if P then
            table.insert(J, P)
        end
    end
elseif Z.Values[o.Default] ~= nil then
    table.insert(J, o.Default)
end
if next(J) then
    for P = 1, #J, 1 do
        local p = J[P]
        if o.Multi then
            Z.Value[Z.Values[p]] = true
        else
            Z.Value = Z.Values[p]
        end
        if not o.Multi then
            break
        end
    end
end
if typeof(Z.Tooltip) == "string" or typeof(Z.DisabledTooltip) == "string" then
    Z.TooltipTable = f:AddTooltip(Z.Tooltip, Z.DisabledTooltip, c)
    Z.TooltipTable.Disabled = Z.Disabled
end
Z:UpdateColors()
Z:Display()
Z:BuildDropdownList()
H:Resize()
Z.Holder = g
table.insert(H.Elements, Z)
Z.Default = J
Z.DefaultValues = Z.Values
z[L] = Z
return Z
end
Y.AddViewport = function(J, L, o)
    o = f:Validate(o, l.Viewport)
    local H = J
    local P, Z, p, g = H.Container, false, false
    local _, c = 0, o.Object
    if o.Clone and typeof(o.Object) == "Instance" then
        J = o.Object
        if J.Archivable then
            c = (c:Clone())
        else
            o.Object.Archivable = true
            c = c:Clone()
            o.Object.Archivable = false
        end
    end
    local e = {Object = c, Camera = not o.Camera and(Instance.new("Camera")) or o.Camera, Interactive = o.Interactive, Zoom = o.Zoom, AutoFocus = o.AutoFocus, Visible = o.Visible, Type = "Viewport"}
    assert(typeof(e.Object) == "Instance" and(e.Object:IsA("BasePart") or(e.Object:IsA("Model"))), "Instance must be a BasePart or Model.")
    assert(typeof(e.Camera) == "Instance" and(e.Camera:IsA("Camera")), "Camera must be a valid Camera instance.")
    local function c(I)
        if I:IsA("BasePart") then return I.Size
    end
    return select(2, I:GetBoundingBox())
end
local function I()
    local x = c(e.Object)
    local c = math.max(x.X, x.Y, x.Z)
    local x, K = c * 2, e.Object:GetPivot().Position
    e.Camera.CFrame = CFrame.new(K + Vector3.new(0, c / 2, x), K)
end
local c = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, o.Height), Visible = e.Visible, Parent = P})
J = F("Frame", {AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 1), Size = UDim2.fromScale(1, 1), Parent = c})
F("UIPadding", {PaddingBottom = UDim.new(0, 3), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 4), Parent = J})
local o = F("ViewportFrame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Parent = J, CurrentCamera = e.Camera, Active = e.Interactive})
o.MouseEnter:Connect(function()
    if not e.Interactive then return
end
for J, J in pairs(H.Tab.Sides) do
    J.ScrollingEnabled = false
end
end)
o.MouseLeave:Connect(function()
    if not e.Interactive then return
end
for J, J in pairs(H.Tab.Sides) do
    J.ScrollingEnabled = true
end
end)
o.InputBegan:Connect(function(J)
    if not e.Interactive then return
end
if J.UserInputType == Enum.UserInputType.MouseButton1 or J.UserInputType == Enum.UserInputType.MouseButton2 then
    Z = true
    g = J.Position
elseif J.UserInputType == Enum.UserInputType.Touch and not p then
    Z = true
    g = J.Position
end
end)
f:GiveSignal(U.InputEnded:Connect(function(J)
    if f.Unloaded then return
end
if not e.Interactive then
    return
end
local P = J.UserInputType == Enum.UserInputType.MouseButton1 or J.UserInputType == Enum.UserInputType.MouseButton2
if P then
    Z = false
else
    local P, x = J.UserInputType, Enum.UserInputType.Touch
    if P == x then
        Z = false
    end
end
end))
f:GiveSignal(U.InputChanged:Connect(function(J)
    if f.Unloaded then return
end
if not e.Interactive or not Z or p then
    return
end
if J.UserInputType == Enum.UserInputType.MouseMovement or J.UserInputType == Enum.UserInputType.Touch then
    local P = J.Position -g
    g = J.Position
    local J, g, x = e.Object:GetPivot().Position, e.Camera, CFrame.fromAxisAngle(Vector3.new(0, 1, 0), -P.X * 0.01)
    g.CFrame = CFrame.new(J) * x * CFrame.new(- J) * g.CFrame
    x = CFrame.fromAxisAngle(g.CFrame.RightVector, -P.Y * 0.01)
    P = CFrame.new(J) * x * CFrame.new(- J) * g.CFrame
    if P.UpVector.Y > 0.1 then
        g.CFrame = P
    end
end
end))
o.InputChanged:Connect(function(J)
    if not e.Interactive or not e.Zoom then return
end
if J.UserInputType == Enum.UserInputType.MouseWheel then
    local P = J.Position.Z * 2
    local J, g, x = e.Camera.CFrame + e.Camera.CFrame.LookVector * P, pcall(function()
        return e.Object:GetPivot().Position
    end)
    if g and x then
        P = (J.Position -x).Magnitude
        if P > 1 and P < 50 then
            e.Camera.CFrame = J
        end
    else
        e.Camera.CFrame = J
    end
end
end)
f:GiveSignal(U.TouchPinch:Connect(function(J, P, g, x)
    if f.Unloaded then return
end
if not e.Interactive or not e.Zoom or not f:MouseIsOverFrame(o, J[1]) then
    return
end
if x == Enum.UserInputState.Begin then
    p, Z = true, false
    _ = (J[1] -J[2]).Magnitude
elseif x == Enum.UserInputState.Change then
    P = (J[1] -J[2]).Magnitude
    g = (P -_) * 0.1
    _ = P
    e.Camera.CFrame = e.Camera.CFrame + e.Camera.CFrame.LookVector * g
else
    local J = x == Enum.UserInputState.End or x == Enum.UserInputState.Cancel
    if J then
        p = false
    end
end
end))
e.Object.Parent = o
if e.AutoFocus then
    I()
end
e.SetObject = function(J, J, P)
    assert(J, "Object cannot be nil.")
    J = if P then(J:Clone()) else J
    if e.Object then
        e.Object:Destroy()
    end
    e.Object = J
    e.Object.Parent = o
    H:Resize()
end
e.SetHeight = function(J, J)
    assert(J > 0, "Height must be greater than 0.")
    c.Size = UDim2.new(1, 0, 0, J)
    H:Resize()
end
e.Focus = function(J)
    if not e.Object then return
end
I()
end
e.SetCamera = function(J, J)
    assert(J and typeof(J) == "Instance" and(J:IsA("Camera")), "Camera must be a valid Camera instance.")
    e.Camera = J
    o.CurrentCamera = J
end
e.SetInteractive = function(J, J)
    e.Interactive = J
    o.Active = J
end
e.SetVisible = function(J, J)
    e.Visible = J
    c.Visible = e.Visible
    H:Resize()
end
H:Resize()
e.Holder = c
table.insert(H.Elements, e)
z[L] = e
return e
end
Y.AddImage = function(J, L, o)
    o = f:Validate(o, l.Image)
    local H = J
    local P, Z = H.Container, {Image = o.Image, Color = o.Color, RectOffset = o.RectOffset, RectSize = o.RectSize, Height = o.Height, ScaleType = o.ScaleType, Transparency = o.Transparency, BackgroundTransparency = o.BackgroundTransparency, Visible = o.Visible, Type = "Image"}
    local p = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, o.Height), Visible = Z.Visible, Parent = P})
    P = F("Frame", {AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, BackgroundTransparency = Z.BackgroundTransparency, Position = UDim2.fromScale(0, 1), Size = UDim2.fromScale(1, 1), Parent = p})
    F("UIPadding", {PaddingBottom = UDim.new(0, 3), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 4), Parent = P})
    J = {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Image = Z.Image, ImageTransparency = Z.Transparency, ImageColor3 = Z.Color, ImageRectOffset = Z.RectOffset, ImageRectSize = Z.RectSize, ScaleType = Z.ScaleType, Parent = P}
    o = f:GetCustomIcon(J.Image)
    assert(o, "Image must be a valid Roblox asset or a valid URL or a valid lucide icon.")
    J.Image = o.Url
    J.ImageRectOffset = o.ImageRectOffset
    J.ImageRectSize = o.ImageRectSize
    local o = F("ImageLabel", J)
    Z.SetHeight = function(J, J)
        assert(J > 0, "Height must be greater than 0.")
        Z.Height = J
        p.Size = UDim2.new(1, 0, 0, J)
        H:Resize()
    end
    Z.SetImage = function(J, P)
        assert(typeof(P) == "string", "Image must be a string.")
        J = f:GetCustomIcon(P)
        assert(J, "Image must be a valid Roblox asset or a valid URL or a valid lucide icon.")
        P = J.Url
        Z.RectOffset = J.ImageRectOffset
        Z.RectSize = J.ImageRectSize
        o.Image = P
        Z.Image = P
    end
    Z.SetColor = function(J, J)
        assert(typeof(J) == "Color3", "Color must be a Color3 value.")
        o.ImageColor3 = J
        Z.Color = J
    end
    Z.SetRectOffset = function(J, J)
        assert(typeof(J) == "Vector2", "RectOffset must be a Vector2 value.")
        o.ImageRectOffset = J
        Z.RectOffset = J
    end
    Z.SetRectSize = function(J, J)
        assert(typeof(J) == "Vector2", "RectSize must be a Vector2 value.")
        o.ImageRectSize = J
        Z.RectSize = J
    end
    Z.SetScaleType = function(J, J)
        assert(typeof(J) == "EnumItem" and(J:IsA("ScaleType")), "ScaleType must be a valid Enum.ScaleType.")
        o.ScaleType = J
        Z.ScaleType = J
    end
    Z.SetTransparency = function(J, J)
        assert(typeof(J) == "number", "Transparency must be a number between 0 and 1.")
        assert(J >= 0 and J <= 1, "Transparency must be between 0 and 1.")
        o.ImageTransparency = J
        Z.Transparency = J
    end
    Z.SetVisible = function(J, J)
        Z.Visible = J
        p.Visible = Z.Visible
        H:Resize()
    end
    H:Resize()
    Z.Holder = p
    table.insert(H.Elements, Z)
    z[L] = Z
    return Z
end
Y.AddVideo = function(J, L, o)
    o = f:Validate(o, l.Video)
    local H = J
    local J, P = H.Container, {Video = o.Video, Looped = o.Looped, Playing = o.Playing, Volume = o.Volume, Height = o.Height, Visible = o.Visible, Type = "Video"}
    local Z = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, o.Height), Visible = P.Visible, Parent = J})
    J = F("Frame", {AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(0, 1), Size = UDim2.fromScale(1, 1), Parent = Z})
    F("UIPadding", {PaddingBottom = UDim.new(0, 3), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 4), Parent = J})
    local o = F("VideoFrame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Video = P.Video, Looped = P.Looped, Volume = P.Volume, Parent = J})
    o.Playing = P.Playing
    P.SetHeight = function(J, J)
        assert(J > 0, "Height must be greater than 0.")
        P.Height = J
        Z.Size = UDim2.new(1, 0, 0, J)
        H:Resize()
    end
    P.SetVideo = function(J, J)
        assert(typeof(J) == "string", "Video must be a string.")
        o.Video = J
        P.Video = J
    end
    P.SetLooped = function(J, J)
        assert(typeof(J) == "boolean", "Looped must be a boolean.")
        o.Looped = J
        P.Looped = J
    end
    P.SetVolume = function(J, J)
        assert(typeof(J) == "number", "Volume must be a number between 0 and 10.")
        o.Volume = J
        P.Volume = J
    end
    P.SetPlaying = function(J, J)
        assert(typeof(J) == "boolean", "Playing must be a boolean.")
        o.Playing = J
        P.Playing = J
    end
    P.Play = function(J)
        o.Playing = true
        P.Playing = true
    end
    P.Pause = function(J)
        o.Playing = false
        P.Playing = false
    end
    P.SetVisible = function(J, J)
        P.Visible = J
        Z.Visible = P.Visible
        H:Resize()
    end
    H:Resize()
    P.Holder = Z
    P.VideoFrame = o
    table.insert(H.Elements, P)
    z[L] = P
    return P
end
Y.AddUIPassthrough = function(J, L, o)
    o = f:Validate(o, l.UIPassthrough)
    local H = J
    J = H.Container
    assert(o.Instance, "Instance must be provided.")
    assert(typeof(o.Instance) == "Instance" and(o.Instance:IsA("GuiBase2d")), "Instance must inherit from GuiBase2d.")
    assert(typeof(o.Height) == "number" and o.Height > 0, "Height must be a number greater than 0.")
    local P = {Instance = o.Instance, Height = o.Height, Visible = o.Visible, Type = "UIPassthrough"}
    local Z = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, o.Height), Visible = P.Visible, Parent = J})
    P.Instance.Parent = Z
    H:Resize()
    P.SetHeight = function(J, J)
        assert(typeof(J) == "number" and J > 0, "Height must be a number greater than 0.")
        P.Height = J
        Z.Size = UDim2.new(1, 0, 0, J)
        H:Resize()
    end
    P.SetInstance = function(J, J)
        assert(J, "Instance must be provided.")
        assert(typeof(J) == "Instance" and(J:IsA("GuiBase2d")), "Instance must inherit from GuiBase2d.")
        if P.Instance then
            P.Instance.Parent = nil
        end
        P.Instance = J
        P.Instance.Parent = Z
    end
    P.SetVisible = function(J, J)
        P.Visible = J
        Z.Visible = P.Visible
        H:Resize()
    end
    P.Holder = Z
    table.insert(H.Elements, P)
    z[L] = P
    return P
end
Y.AddDependencyBox = function(J)
    local L = J
    local J, o, H = L.Container
    do
        o = F("Frame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Visible = false, Parent = J})
        H = F("UIListLayout", {Padding = UDim.new(0, 8), Parent = o})
    end
    local J
    J = {Visible = false, Dependencies = {}, Holder = o, Container = o, Elements = {}, DependencyBoxes = {}, Resize = function(P)
        o.Size = UDim2.new(1, 0, 0, H.AbsoluteContentSize.Y / f.DPIScale)
        L:Resize()
    end, Update = function(P, Z)
        for p, g in pairs(J.Dependencies) do
            p, P = g[1], g[2]
            if p.Type == "Toggle" and p.Value ~= P then
                o.Visible = false
                J.Visible = false
                return
            elseif p.Type == "Dropdown" then
                if typeof(p.Value) == "table" then
                    if not p.Value[P] then
                        o.Visible = false
                        J.Visible = false
                        return
                    end
                elseif p.Value ~= P then
                    o.Visible = false
                    J.Visible = false
                    return
                end
            end
        end
        J.Visible = true
        o.Visible = true
        if not f.Searching then
            task.defer(function()
                J:Resize()
            end)
        elseif not Z then
            f:UpdateSearch(f.SearchText)
        end
    end
    }
    H:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        if not J.Visible then return
    end
    J:Resize()
end)
J.SetupDependencies = function(H, H)
    for P, P in pairs(H) do
        assert(typeof(P) == "table", "Dependency should be a table.")
        assert(P[1] ~= nil, "Dependency is missing element.")
        assert(P[2] ~= nil, "Dependency is missing expected value.")
    end
    J.Dependencies = H
    J:Update()
end
o:GetPropertyChangedSignal("Visible"):Connect(function()
    J:Resize()
end)
setmetatable(J, s)
J.Destroy = function(H)
    for P = #J.DependencyBoxes, 1, -1 do
        H = J.DependencyBoxes[P]
        if H and H.Destroy then
            H:Destroy()
        end
    end
    J:Clear()
    for H = #L.DependencyBoxes, 1, -1 do
        if L.DependencyBoxes[H] == J then
            table.remove(L.DependencyBoxes, H)
        end
    end
    for H = #f.DependencyBoxes, 1, -1 do
        if f.DependencyBoxes[H] == J then
            table.remove(f.DependencyBoxes, H)
        end
    end
    pcall(function()
        o:Destroy()
    end)
    L:Resize()
end
table.insert(L.DependencyBoxes, J)
table.insert(f.DependencyBoxes, J)
return J
end
Y.AddDependencyGroupbox = function(J)
    local L, o, H, P = J.Tab, J.BoxHolder
    do
        H = F("Frame", {BackgroundColor3 = "BackgroundColor", Size = UDim2.fromScale(1, 0), Visible = false, Parent = o})
        F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = H})
        f:AddOutline(H)
        P = F("UIListLayout", {Padding = UDim.new(0, 8), Parent = H})
        F("UIPadding", {PaddingBottom = UDim.new(0, 7), PaddingLeft = UDim.new(0, 7), PaddingRight = UDim.new(0, 7), PaddingTop = UDim.new(0, 7), Parent = H})
    end
    local J
    J = {Visible = false, Dependencies = {}, BoxHolder = o, Holder = H, Container = H, Tab = L, Elements = {}, DependencyBoxes = {}, Resize = function(o)
        H.Size = UDim2.new(1, 0, 0, P.AbsoluteContentSize.Y / f.DPIScale + 18)
    end, Update = function(o, P)
        for Z, p in pairs(J.Dependencies) do
            o, Z = p[1], p[2]
            if o.Type == "Toggle" and o.Value ~= Z then
                H.Visible = false
                J.Visible = false
                return
            elseif o.Type == "Dropdown" then
                if typeof(o.Value) == "table" then
                    if not o.Value[Z] then
                        H.Visible = false
                        J.Visible = false
                        return
                    end
                elseif o.Value ~= Z then
                    H.Visible = false
                    J.Visible = false
                    return
                end
            end
        end
        J.Visible = true
        if not f.Searching then
            H.Visible = true
            J:Resize()
        elseif not P then
            f:UpdateSearch(f.SearchText)
        end
    end, SetupDependencies = function(o, o)
        for H, H in pairs(o) do
            assert(typeof(H) == "table", "Dependency should be a table.")
            assert(H[1] ~= nil, "Dependency is missing element.")
            assert(H[2] ~= nil, "Dependency is missing expected value.")
        end
        J.Dependencies = o
        J:Update()
    end
    }
    setmetatable(J, s)
    table.insert(L.DependencyGroupboxes, J)
    table.insert(f.DependencyBoxes, J)
    return J
end
s.__index = Y
s.__namecall = function(J, J, ...)
    return Y[J](...)
end
f.BaseGroupbox = s
end
f.SetFont = function(J, L)
    J = typeof(L)
    f.Scheme.Font = if J == "EnumItem" then(Font.fromEnum(L)) else L
    f:UpdateColorsUsingRegistry()
end
f.SetNotifySide = function(J, J)
    f.NotifySide = J
end
f.Notify = function(J, ...)
    local L, o = {}, select(1, ...)
    if typeof(o) == "table" then
        L.Title = tostring(o.Title)
        L.Description = tostring(o.Description)
        L.Time = o.Time or 5
        L.SoundId = o.SoundId
        L.Steps = o.Steps
        L.Persist = o.Persist
    else
        L.Description = tostring(o)
        L.Time = select(2, ...) or 5
        L.SoundId = select(3, ...)
    end
    L.Destroyed = false
    local H, P = false
    if typeof(L.Time) == "Instance" then
        P = L.Time.Destroying:Connect(function()
            H = true
            P:Disconnect()
            P = nil
        end)
    end
    local Y = F("Frame", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), Visible = false, Parent = D})
    local Z = F("Frame", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundColor3 = "MainColor", Position = f.NotifySide:lower() == "left" and(UDim2.new(- 1, -8, 0, -2)) or(UDim2.new(1, 8, 0, -2)), Size = UDim2.fromScale(1, 1), ZIndex = 5, Parent = Y})
    F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = Z})
    F("UIListLayout", {Padding = UDim.new(0, 4), Parent = Z})
    F("UIPadding", {PaddingBottom = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 8), Parent = Z})
    f:AddOutline(Z)
    local p, g
    local _, c, e = 0, 0
    if L.Title then
        p = F("TextLabel", {AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Size = UDim2.fromScale(0, 0), Text = L.Title, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true, Parent = Z})
    end
    if L.Description then
        g = F("TextLabel", {AutomaticSize = Enum.AutomaticSize.X, BackgroundTransparency = 1, Size = UDim2.fromScale(0, 0), Text = L.Description, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, TextWrapped = true, Parent = Z})
    end
    L.Resize = function(I)
        if p then local I, x = f:GetTextBounds(p.Text, p.FontFace, p.TextSize, D.AbsoluteSize.X / f.DPIScale -24)
        p.Size = UDim2.fromOffset(0, x)
        _ = I
    end
    if g then
        local I, x = f:GetTextBounds(g.Text, g.FontFace, g.TextSize, D.AbsoluteSize.X / f.DPIScale -24)
        g.Size = UDim2.fromOffset(0, x)
        c = I
    end
    Y.Size = UDim2.fromOffset(math.max(_, c) + 24, 0)
end
L.ChangeTitle = function(_, _)
    if p then L.Title = tostring(_)
    p.Text = L.Title
    L:Resize()
end
end
L.ChangeDescription = function(p, p)
    if g then L.Description = tostring(p)
    g.Text = L.Description
    L:Resize()
end
end
L.ChangeStep = function(p, p)
    if e and L.Steps then p = math.clamp(p or 0, 0, L.Steps)
    e.Size = UDim2.fromScale(p / L.Steps, 1)
end
end
L.Destroy = function(p)
    L.Destroyed = true
    if typeof(L.Time) == "Instance" then
        pcall(L.Time.Destroy, L.Time)
    end
    if P then
        P:Disconnect()
    end
    B:Create(Z, f.NotifyTweenInfo, {Position = f.NotifySide:lower() == "left" and(UDim2.new(- 1, -8, 0, -2)) or(UDim2.new(1, 8, 0, -2))}):Play()
    task.delay(f.NotifyTweenInfo.Time, function()
        f.Notifications[Y] = nil
        Y:Destroy()
    end)
end
L:Resize()
o = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 7), Visible = L.Persist ~= true and typeof(L.Time) ~= "Instance" or typeof(L.Steps) == "number", Parent = Z})
J = F("Frame", {BackgroundColor3 = "BackgroundColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromOffset(0, 3), Size = UDim2.new(1, 0, 0, 2), Parent = o})
e = F("Frame", {BackgroundColor3 = "AccentColor", Size = UDim2.fromScale(1, 1), Parent = J})
if typeof(L.Time) == "Instance" then
    e.Size = UDim2.fromScale(0, 1)
end
if L.SoundId then
    o = L.SoundId
    F("Sound", {SoundId = if typeof(o) == "number" then(string.format("rbxassetid://%d", o)) else o, Volume = 3, PlayOnRemove = true, Parent = W}):Destroy()
end
f.Notifications[Y] = L
Y.Visible = true
B:Create(Z, f.NotifyTweenInfo, {Position = UDim2.fromOffset(0, 0)}):Play()
task.delay(f.NotifyTweenInfo.Time, function()
    if L.Persist then return
elseif typeof(L.Time) == "Instance" then
    repeat
        task.wait()
        until H or L.Destroyed
    else
        B:Create(e, TweenInfo.new(L.Time, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut), {Size = UDim2.fromScale(0, 1)}):Play()
        task.wait(L.Time)
    end
    if not L.Destroyed then
        L:Destroy()
    end
end)
return L
end
f.CreateWindow = function(J, W)
    W = f:Validate(W, l.Window)
    J = workspace.CurrentCamera.ViewportSize
    if q:IsStudio() and J.X <= 5 and J.Y <= 5 then
        repeat
            J = workspace.CurrentCamera.ViewportSize
            task.wait()
            until J.X > 5 and J.Y > 5
        end
        local L, o = J.X -64, J.Y -64
        f.OriginalMinSize = Vector2.new(math.min(f.OriginalMinSize.X, L), math.min(f.OriginalMinSize.Y, o))
        f.MinSize = f.OriginalMinSize
        W.Size = UDim2.fromOffset(math.clamp(W.Size.X.Offset, f.MinSize.X, L), math.clamp(W.Size.Y.Offset, f.MinSize.Y, o))
        if typeof(W.Font) == "EnumItem" then
            W.Font = Font.fromEnum(W.Font)
        end
        W.CornerRadius = math.min(W.CornerRadius, 20)
        if W.Compact ~= nil then
            W.SidebarCompacted = W.Compact
        end
        if W.SidebarMinWidth ~= nil then
            W.MinSidebarWidth = W.SidebarMinWidth
        end
        W.MinSidebarWidth = math.max(64, W.MinSidebarWidth)
        W.SidebarCompactWidth = math.max(48, W.SidebarCompactWidth)
        W.SidebarCollapseThreshold = math.clamp(W.SidebarCollapseThreshold, 0.1, 0.9)
        W.CompactWidthActivation = math.max(48, W.CompactWidthActivation)
        f.CornerRadius = W.CornerRadius
        f:SetNotifySide(W.NotifySide)
        f.ShowCustomCursor = W.ShowCustomCursor
        f.Scheme.Font = W.Font
        f.ToggleKeybind = W.ToggleKeybind
        f.GlobalSearch = W.GlobalSearch
        local H, P, Y, l, Z, p, g, _, c, e, I, x, D, K, j, O, m = W.SearchbarSize == UDim2.fromScale(1, 1)
        local k, n, E, S = false, false, {}
        local G, y, i, T = 1, {}, math.ceil(W.Size.X.Offset * 0.3), W.SidebarCompacted
        local MJ = i
        do
            f.KeybindFrame, f.KeybindContainer = f:AddDraggableMenu("Keybinds")
            f.KeybindFrame.AnchorPoint = Vector2.new(0, 0.5)
            f.KeybindFrame.Position = UDim2.new(0, 6, 0.5, 0)
            f.KeybindFrame.Visible = false
            P = F("TextButton", {BackgroundColor3 = function()
                return f:GetBetterColor(f.Scheme.BackgroundColor, -1)
            end, Name = "Main", Text = "", Position = W.Position, Size = W.Size, Visible = false, ClipsDescendants = true, BackgroundTransparency = W.GlassType == "Full" and 0.3 or(W.GlassType == "TopOnly" and 1 or 0), Parent = b})
            f._customBg = F("ImageLabel", {BackgroundTransparency = 1, Image = "", ImageTransparency = 1, Size = UDim2.fromScale(1, 1), ScaleType = Enum.ScaleType.Crop, ZIndex = 0, Visible = false, Parent = P})
            F("UICorner", {CornerRadius = UDim.new(0, f.CornerRadius), Parent = f._customBg})
            o = F("UIStroke", {Color = function()
                return f:GetBetterColor(f.Scheme.OutlineColor, 2)
            end, Transparency = 0.5, Thickness = 1, Parent = P})
            f.Registry[o].Color = function()
                return f:GetBetterColor(f.Scheme.OutlineColor, 2)
            end
            if W.GlassType == "Full" then
                S = F("UIStroke", {Color = Color3.fromRGB(255, 255, 255), Transparency = 0.9, Thickness = 1, Parent = P})
            end
            j = F("Frame", {BackgroundColor3 = Color3.fromRGB(0, 0, 0), BackgroundTransparency = 1, Size = UDim2.new(2, 0, 2, 0), Position = UDim2.fromScale(- 0.5, -0.5), ZIndex = -1, Visible = false, Parent = b})
            O = F("BlurEffect", {Size = 0, Parent = game:GetService("Lighting")})
            for o, o in pairs(game:GetService("Lighting"):GetChildren()) do
                if o:IsA("BlurEffect") and o ~= O then
                    o:Destroy()
                end
            end
            if W.EnableSnowfall then
                k = true
                m = F("Frame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), ZIndex = 10, Visible = W.AutoShow, Parent = b})
                n = W.AutoShow
                local o, bJ = game:GetService("RunService"), {}
                for JJ = 1, 100, 1 do
                    local NJ, AJ, qJ, WJ = math.random(3, 6), math.random(30, 60) / 100, math.random(0, 100) / 100, math.random(- 50, -5) / 100
                    L = F("Frame", {BackgroundColor3 = Color3.fromRGB(255, 255, 255), BackgroundTransparency = AJ, Size = UDim2.fromOffset(NJ, NJ), Position = UDim2.fromScale(qJ, WJ), Rotation = math.random(0, 360), Parent = m})
                    F("UICorner", {CornerRadius = UDim.new(1, 0), Parent = L})
                    bJ[JJ] = {frame = L, velY = 0.012, velX = math.random(- 3, 3) / 1000, rotSpeed = math.random(- 60, 60) / 100, delay = math.random(0, 150) / 100, elapsed = 0, posY = WJ, posX = qJ, rot = math.random(0, 360)}
                end
                f:GiveSignal((o.Heartbeat:Connect(function(o)
                    if not n then return
                end
                local JJ = o * 60
                for NJ, NJ in pairs(bJ) do
                    if NJ.elapsed < NJ.delay then
                        NJ.elapsed = NJ.elapsed + o
                    else
                        NJ.posY = NJ.posY + NJ.velY * G * JJ
                        NJ.posX = NJ.posX + NJ.velX * JJ
                        NJ.rot = NJ.rot + NJ.rotSpeed * JJ
                        if NJ.posY > 1.2 then
                            NJ.posY = math.random(- 30, -5) / 100
                            NJ.posX = math.random(0, 100) / 100
                        end
                        if NJ.posX < -0.05 then
                            NJ.posX = 1.05
                        end
                        if NJ.posX > 1.05 then
                            NJ.posX = -0.05
                        end
                        NJ.frame.Position = UDim2.fromScale(NJ.posX, NJ.posY)
                        NJ.frame.Rotation = NJ.rot
                    end
                end
            end)))
        end
        if W.EnableFadeIn then
            P.Size = UDim2.fromOffset(W.Size.X.Offset * 0.95, W.Size.Y.Offset * 0.95)
            P.BackgroundTransparency = 1
            task.spawn(function()
                task.wait(0.05)
                local o = W.GlassType == "Full" and 0.3 or(W.GlassType == "TopOnly" and 1 or 0)
                B:Create(P, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = W.Size, BackgroundTransparency = o}):Play()
            end)
        end
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = P})
        table.insert(f.Scales, F("UIScale", {Parent = P}))
        f:AddOutline(P)
        f:MakeLine(P, {Position = UDim2.fromOffset(0, 48), Size = UDim2.new(1, 0, 0, 1)})
        Y = F("Frame", {BackgroundColor3 = "OutlineColor", Position = UDim2.fromOffset(i, 0), Size = UDim2.new(0, 1, 1, -21), Parent = P})
        if W.BackgroundImage then
            F("ImageLabel", {Image = W.BackgroundImage, Position = UDim2.fromScale(0, 0), Size = UDim2.fromScale(1, 1), ScaleType = Enum.ScaleType.Stretch, ZIndex = 999, BackgroundTransparency = 1, ImageTransparency = 0.75, Parent = P})
        end
        if W.Center then
            P.AnchorPoint = Vector2.new(0.5, 0.5)
            P.Position = UDim2.fromScale(0.5, 0.5)
        end
        J = F("Frame", {Name = "TopBar", BackgroundTransparency = W.GlassType == "TopOnly" and 0.3 or 1, BackgroundColor3 = function()
            return f:GetBetterColor(f.Scheme.BackgroundColor, -1)
        end, Size = UDim2.new(1, 0, 0, 48), Parent = P})
        if W.GlassType == "TopOnly" then
            F("UIStroke", {Color = Color3.fromRGB(255, 255, 255), Transparency = 0.9, Thickness = 1, Parent = J})
            F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = J})
        end
        f:MakeDraggable(P, J, false, true)
        l = F("Frame", {BackgroundTransparency = 1, ClipsDescendants = true, Size = UDim2.new(0, i, 1, 0), Parent = J})
        if W.Icon then
            p = F("ImageLabel", {AnchorPoint = Vector2.new(0, 0.5), Position = UDim2.new(0, 14, 0.5, 0), Image = tonumber(W.Icon) and(string.format("rbxassetid://%d", W.Icon)) or W.Icon, Size = W.IconSize, Parent = l})
        else
            p = F("TextLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, Position = UDim2.fromScale(0.5, 0.5), Size = W.IconSize, Text = W.Title:sub(1, 1), TextScaled = true, Visible = false, Parent = l})
        end
        Z = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = W.Title, TextSize = 20, TextXAlignment = Enum.TextXAlignment.Center, Parent = l})
        g = F("Frame", {AnchorPoint = Vector2.new(1, 0.5), BackgroundTransparency = 1, Position = UDim2.new(1, -49, 0.5, 0), Size = UDim2.new(1, -i -57 -1, 1, -16), Parent = J})
        F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Left, VerticalAlignment = Enum.VerticalAlignment.Center, Padding = UDim.new(0, 8), Parent = g})
        c = F("Frame", {Size = UDim2.fromScale(W.DisableSearch and 1 or 0.5, 1), Visible = false, BackgroundTransparency = 1, Parent = g})
        F("UIFlexItem", {FlexMode = Enum.UIFlexMode.Grow, Parent = c})
        F("UIListLayout", {FillDirection = Enum.FillDirection.Vertical, HorizontalAlignment = Enum.HorizontalAlignment.Left, VerticalAlignment = Enum.VerticalAlignment.Center, Parent = c})
        F("UIPadding", {PaddingBottom = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 8), Parent = c})
        e = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), AutomaticSize = Enum.AutomaticSize.Y, Text = "", TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = c})
        I = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), AutomaticSize = Enum.AutomaticSize.Y, Text = "", TextWrapped = true, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, TextTransparency = 0.5, Parent = c})
        _ = F("TextBox", {BackgroundColor3 = "MainColor", PlaceholderText = "Search", Size = W.SearchbarSize, TextScaled = true, Visible = not(W.DisableSearch or false), Parent = g})
        F("UIFlexItem", {FlexMode = Enum.UIFlexMode.Shrink, Parent = _})
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = _})
        F("UIPadding", {PaddingBottom = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 8), Parent = _})
        F("UIStroke", {Color = "OutlineColor", Parent = _})
        local o = f:GetIcon("search")
        if o then
            F("ImageLabel", {Image = o.Url, ImageColor3 = "FontColor", ImageRectOffset = o.ImageRectOffset, ImageRectSize = o.ImageRectSize, ImageTransparency = 0.5, Size = UDim2.fromScale(1, 1), SizeConstraint = Enum.SizeConstraint.RelativeYY, Parent = _})
        end
        if V then
            F("ImageLabel", {AnchorPoint = Vector2.new(1, 0.5), Image = V.Url, ImageColor3 = "OutlineColor", ImageRectOffset = V.ImageRectOffset, ImageRectSize = V.ImageRectSize, Position = UDim2.new(1, -10, 0.5, 0), Size = UDim2.fromOffset(28, 28), SizeConstraint = Enum.SizeConstraint.RelativeYY, Parent = J})
        end
        o = F("Frame", {AnchorPoint = Vector2.new(0, 1), BackgroundColor3 = function()
            return f:GetBetterColor(f.Scheme.BackgroundColor, 4)
        end, Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 20), Parent = P})
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = o})
        F("Frame", {BackgroundColor3 = function()
            return f:GetBetterColor(f.Scheme.BackgroundColor, 4)
        end, BorderSizePixel = 0, Size = UDim2.new(1, 0, 0, W.CornerRadius), Parent = o})
        f:MakeLine(P, {AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 0, 1, -20), Size = UDim2.new(1, 0, 0, 1)})
        o = F("Frame", {AnchorPoint = Vector2.new(0, 1), BackgroundTransparency = 1, Position = UDim2.fromScale(0, 1), Size = UDim2.new(1, 0, 0, 20), Parent = P})
        F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Text = W.Footer, TextSize = 14, TextTransparency = 0.5, Parent = o})
        if W.Resizable then
            x = F("TextButton", {AnchorPoint = Vector2.new(1, 0), BackgroundTransparency = 1, Position = UDim2.new(1, -W.CornerRadius / 4, 0, 0), Size = UDim2.fromScale(1, 1), SizeConstraint = Enum.SizeConstraint.RelativeYY, Text = "", Parent = o})
            f:MakeResizable(P, x, function()
                W.Size = P.Size
                f._persistentWindowSize = P.Size
                for o, o in pairs(f.Tabs) do
                    o:Resize(true)
                end
            end)
            f._persistentWindowSize = W.Size
        end
        F("ImageLabel", {Image = v and v.Url or "", ImageColor3 = "FontColor", ImageRectOffset = v and v.ImageRectOffset or Vector2.zero, ImageRectSize = v and v.ImageRectSize or Vector2.zero, ImageTransparency = 0.5, Position = UDim2.fromOffset(2, 2), Size = UDim2.new(1, -4, 1, -4), Parent = x})
        D = F("ScrollingFrame", {AutomaticCanvasSize = Enum.AutomaticSize.Y, BackgroundColor3 = "BackgroundColor", CanvasSize = UDim2.fromScale(0, 0), ClipsDescendants = true, Position = UDim2.fromOffset(0, 49), ScrollBarThickness = 0, Size = UDim2.new(0, i, 1, -70), Parent = P})
        table.insert(y, D)
        F("UIListLayout", {Parent = D})
        K = F("Frame", {AnchorPoint = Vector2.new(1, 0), BackgroundColor3 = function()
            return f:GetBetterColor(f.Scheme.BackgroundColor, 1)
        end, Name = "Container", Position = UDim2.new(1, 0, 0, 49), Size = UDim2.new(1, -i -1, 1, -70), Parent = P})
        table.insert(y, K)
        F("UIPadding", {PaddingBottom = UDim.new(0, 0), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6), PaddingTop = UDim.new(0, 0), Parent = K})
    end
    local o = {Frame = P}
    f.Window = o
    o.ChangeTitle = function(v, v)
        assert(typeof(v) == "string", "Expected string for title got: " .. typeof(v))
        Z.Text = v
        W.Title = v
    end
    local function v()
        T = o:GetSidebarWidth() == W.SidebarCompactWidth
        local x = W
        if x.DisableCompactingSnap then
            T = o:GetSidebarWidth() <= W.CompactWidthActivation
        end
        Z.Visible = not T
        if not W.Icon then
            p.Visible = T
        end
        for p, p in pairs(f.TabButtons) do
            if p.Icon then
                p.Label.Visible = not T
                p.Padding.PaddingBottom = UDim.new(0, T and 6 or 11)
                p.Padding.PaddingLeft = UDim.new(0, T and 6 or 12)
                p.Padding.PaddingRight = UDim.new(0, T and 6 or 12)
                p.Padding.PaddingTop = UDim.new(0, T and 6 or 11)
                p.Icon.SizeConstraint = T and Enum.SizeConstraint.RelativeXY or Enum.SizeConstraint.RelativeYY
            end
        end
    end
    o.IsSidebarCompacted = function(p)
        return T
    end
    o.SetCompact = function(p, p)
        o:SetSidebarWidth(p and W.SidebarCompactWidth or MJ)
    end
    o.GetSidebarWidth = function(p)
        return D.Size.X.Offset
    end
    o.SetSidebarWidth = function(p, x)
        x = math.clamp(x, 48, P.Size.X.Offset -W.MinContainerWidth -1)
        Y.Position = UDim2.fromOffset(x, 0)
        l.Size = UDim2.new(0, x, 1, 0)
        g.Size = UDim2.new(1, -x -57 -1, 1, -16)
        D.Size = UDim2.new(0, x, 1, -70 -(o._cardReserve or 0))
        K.Size = UDim2.new(1, -x -1, 1, -70)
        if W.EnableCompacting then
            v()
        end
        p = T
        if not p then
            MJ = x
        end
        if o._onSidebarWidth then
            o._onSidebarWidth(x)
        end
    end
    o.ShowTabInfo = function(v, v, p)
        e.Text = v
        I.Text = p
        if H then
            _.Size = UDim2.fromScale(0.5, 1)
        end
        c.Visible = true
    end
    o.HideTabInfo = function(v)
        c.Visible = false
        if H then
            _.Size = UDim2.fromScale(1, 1)
        end
    end
    local H, v, p, g, c
    o.SetPlayerCard = function(e, e)
        e = e or{}
        local I, x, V = tonumber(e.UserId) or A.LocalPlayer and A.LocalPlayer.UserId or 1, e.Username or A.LocalPlayer and A.LocalPlayer.DisplayName or A.LocalPlayer and A.LocalPlayer.Name or "Player", e.Status or ""
        if not H then
            H = F("Frame", {BackgroundColor3 = "MainColor", Position = UDim2.new(0, 8, 1, -70), Size = UDim2.new(0, 100, 0, 44), Parent = P})
            F("UICorner", {CornerRadius = UDim.new(0, 6), Parent = H})
            F("UIStroke", {Color = "OutlineColor", Transparency = 0.4, Parent = H})
            v = F("ImageLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundColor3 = "OutlineColor", Position = UDim2.new(0, 22, 0.5, 0), Size = UDim2.fromOffset(32, 32), Parent = H})
            F("UICorner", {CornerRadius = UDim.new(1, 0), Parent = v})
            p = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.new(0, 46, 0, 6), Size = UDim2.new(1, -52, 0, 16), Text = "", TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Parent = H})
            g = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.new(0, 46, 0, 23), Size = UDim2.new(1, -52, 0, 14), Text = "", TextColor3 = Color3.new(1, 1, 1), TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Parent = H})
            c = F("UIGradient", {Parent = g})
            local MJ, bJ = 0
            bJ = q.RenderStepped:Connect(function(JJ)
                if f.Unloaded or not c.Parent then
                if bJ then
                    bJ:Disconnect()
                end
                return
            end
            MJ = (MJ + JJ * 1.5) % 6.2831853
            c.Offset = Vector2.new(math.sin(MJ), 0)
        end)
        o._cardReserve = 0
        function o._onSidebarWidth(MJ)
            local bJ = math.max(32, MJ -16)
            H.Size = UDim2.new(0, bJ, 0, 44)
            local H = 26 + 6 * math.clamp((MJ -54) / 120, 0, 1)
            v.Size = UDim2.fromOffset(H, H)
            v.Position = UDim2.new(0, math.min(bJ / 2, 8 + H / 2), 0.5, 0)
            H = math.clamp((MJ -100) / 60, 0, 1)
            p.TextTransparency = 1 -H
            g.TextTransparency = g.Text ~= "" and 1 -H or 1
        end
        o:SetSidebarWidth(o:GetSidebarWidth())
    end
    v.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", I)
    p.Text = x
    g.Text = V
    c.Color = e.StatusGradient or(ColorSequence.new({ColorSequenceKeypoint.new(0, f.Scheme.AccentColor), ColorSequenceKeypoint.new(1, f.Scheme.FontColor)}))
    if o._onSidebarWidth then
        o._onSidebarWidth(o:GetSidebarWidth())
    end
end
o.AddTab = function(H, ...)
    local v, p, g
    if select("#", ...) == 1 and typeof(...) == "table" then
        H = select(1, ...)
        v = H.Name or "Tab"
        g, p = H.Icon, H.Description
    else
        v = select(1, ...)
        g, p = select(2, ...), select(3, ...)
    end
    local c, e, I, x, V, MJ
    g = f:GetCustomIcon(g)
    do
        c = F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 40), Text = "", Parent = D})
        F("UIGradient", {Color = ColorSequence.new({ColorSequenceKeypoint.new(0, f.Scheme.AccentColor), ColorSequenceKeypoint.new(0.5, Color3.fromRGB(math.clamp(f.Scheme.AccentColor.R * 255 + 30, 0, 255), math.clamp(f.Scheme.AccentColor.G * 255 + 30, 0, 255), math.clamp(f.Scheme.AccentColor.B * 255 + 30, 0, 255))), ColorSequenceKeypoint.new(1, f.Scheme.AccentColor)}), Rotation = 45, Enabled = false, Parent = c})
        H = F("UIPadding", {PaddingBottom = UDim.new(0, T and 6 or 11), PaddingLeft = UDim.new(0, T and 6 or 12), PaddingRight = UDim.new(0, T and 6 or 12), PaddingTop = UDim.new(0, T and 6 or 11), Parent = c})
        e = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(30, 0), Size = UDim2.new(1, -30, 1, 0), Text = v, TextSize = 16, TextTransparency = 0.5, TextXAlignment = Enum.TextXAlignment.Left, Visible = not T, Parent = c})
        if g then
            I = F("ImageLabel", {Image = g.Url, ImageColor3 = g.Custom and "WhiteColor" or "AccentColor", ImageRectOffset = g.ImageRectOffset, ImageRectSize = g.ImageRectSize, ImageTransparency = 0.5, ScaleType = Enum.ScaleType.Fit, Size = UDim2.fromScale(1, 1), SizeConstraint = T and Enum.SizeConstraint.RelativeXY or Enum.SizeConstraint.RelativeYY, Parent = c})
        end
        table.insert(f.TabButtons, {Label = e, Padding = H, Icon = I})
        x = F("Frame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), Position = UDim2.fromOffset(0, 0), Visible = false, Parent = K})
        V = F("ScrollingFrame", {AutomaticCanvasSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, CanvasSize = UDim2.fromScale(0, 0), ScrollBarImageTransparency = 1, ScrollBarThickness = 0, Size = UDim2.new(0.5, -3, 1, 0), Parent = x})
        F("UIListLayout", {Padding = UDim.new(0, 2), Parent = V})
        F("UIPadding", {PaddingBottom = UDim.new(0, 2), PaddingLeft = UDim.new(0, 2), PaddingRight = UDim.new(0, 2), PaddingTop = UDim.new(0, 2), Parent = V})
        F("Frame", {BackgroundTransparency = 1, LayoutOrder = -1, Parent = V})
        F("Frame", {BackgroundTransparency = 1, LayoutOrder = 1, Parent = V})
        MJ = F("ScrollingFrame", {AnchorPoint = Vector2.new(1, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, CanvasSize = UDim2.fromScale(0, 0), Position = UDim2.fromScale(1, 0), ScrollBarImageTransparency = 1, ScrollBarThickness = 0, Size = UDim2.new(0.5, -3, 1, 0), Parent = x})
        F("UIListLayout", {Padding = UDim.new(0, 2), Parent = MJ})
        F("UIPadding", {PaddingBottom = UDim.new(0, 2), PaddingLeft = UDim.new(0, 2), PaddingRight = UDim.new(0, 2), PaddingTop = UDim.new(0, 2), Parent = MJ})
        F("Frame", {BackgroundTransparency = 1, LayoutOrder = -1, Parent = MJ})
        F("Frame", {BackgroundTransparency = 1, LayoutOrder = 1, Parent = MJ})
    end
    local H = F("Frame", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 7), Size = UDim2.fromScale(1, 0), Visible = false, Parent = x})
    local g, bJ, JJ, NJ, AJ, qJ, WJ
    do
        g = F("Frame", {BackgroundColor3 = "BackgroundColor", Position = UDim2.fromOffset(2, 0), Size = UDim2.new(1, -5, 0, 0), Parent = H})
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = g})
        bJ, JJ = f:AddOutline(g)
        NJ = F("ScrollingFrame", {BackgroundTransparency = 1, BorderSizePixel = 0, Size = UDim2.fromScale(1, 1), CanvasSize = UDim2.new(0, 0, 0, 0), ScrollBarThickness = 3, ScrollingDirection = Enum.ScrollingDirection.Y, Parent = g})
        F("UIPadding", {PaddingBottom = UDim.new(0, 4), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 6), PaddingTop = UDim.new(0, 4), Parent = NJ})
        AJ = F("TextLabel", {BackgroundTransparency = 1, Size = UDim2.new(1, -4, 0, 14), Text = "", TextColor3 = Color3.fromRGB(255, 50, 50), TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = NJ})
        qJ = F("UIStroke", {ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual, Color = Color3.fromRGB(169, 0, 0), LineJoinMode = Enum.LineJoinMode.Miter, Parent = AJ})
        WJ = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 16), Size = UDim2.new(1, -4, 0, 0), Text = "", TextSize = 14, TextWrapped = true, Parent = NJ, TextXAlignment = Enum.TextXAlignment.Left, TextYAlignment = Enum.TextYAlignment.Top})
        F("UIStroke", {ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual, Color = "DarkColor", LineJoinMode = Enum.LineJoinMode.Miter, Parent = WJ})
    end
    local UJ
    UJ = {Groupboxes = {}, Tabboxes = {}, DependencyGroupboxes = {}, Sides = {V, MJ}, WarningBox = {IsNormal = false, LockSize = false, Visible = false, Title = "WARNING", Text = ""}, UpdateWarningBox = function(LJ, LJ)
        if typeof(LJ.IsNormal) == "boolean" then UJ.WarningBox.IsNormal = LJ.IsNormal
    end
    if typeof(LJ.LockSize) == "boolean" then
        UJ.WarningBox.LockSize = LJ.LockSize
    end
    if typeof(LJ.Visible) == "boolean" then
        UJ.WarningBox.Visible = LJ.Visible
    end
    if typeof(LJ.Title) == "string" then
        UJ.WarningBox.Title = LJ.Title
    end
    if typeof(LJ.Text) == "string" then
        UJ.WarningBox.Text = LJ.Text
    end
    H.Visible = UJ.WarningBox.Visible
    AJ.Text = UJ.WarningBox.Title
    WJ.Text = UJ.WarningBox.Text
    UJ:Resize(true)
    g.BackgroundColor3 = UJ.WarningBox.IsNormal == true and f.Scheme.BackgroundColor or(Color3.fromRGB(127, 0, 0))
    JJ.Color = UJ.WarningBox.IsNormal == true and f.Scheme.DarkColor or(Color3.fromRGB(85, 0, 0))
    bJ.Color = UJ.WarningBox.IsNormal == true and f.Scheme.OutlineColor or(Color3.fromRGB(255, 50, 50))
    AJ.TextColor3 = UJ.WarningBox.IsNormal == true and f.Scheme.FontColor or(Color3.fromRGB(255, 50, 50))
    qJ.Color = UJ.WarningBox.IsNormal == true and f.Scheme.OutlineColor or(Color3.fromRGB(169, 0, 0))
    if not f.Registry[g] then
        f:AddToRegistry(g, {})
    end
    if not f.Registry[JJ] then
        f:AddToRegistry(JJ, {})
    end
    if not f.Registry[bJ] then
        f:AddToRegistry(bJ, {})
    end
    if not f.Registry[AJ] then
        f:AddToRegistry(AJ, {})
    end
    if not f.Registry[qJ] then
        f:AddToRegistry(qJ, {})
    end
    f.Registry[g].BackgroundColor3 = function()
        return UJ.WarningBox.IsNormal == true and f.Scheme.BackgroundColor or(Color3.fromRGB(127, 0, 0))
    end
    f.Registry[JJ].Color = function()
        return UJ.WarningBox.IsNormal == true and f.Scheme.DarkColor or(Color3.fromRGB(85, 0, 0))
    end
    f.Registry[bJ].Color = function()
        return UJ.WarningBox.IsNormal == true and f.Scheme.OutlineColor or(Color3.fromRGB(255, 50, 50))
    end
    f.Registry[AJ].TextColor3 = function()
        return UJ.WarningBox.IsNormal == true and f.Scheme.FontColor or(Color3.fromRGB(255, 50, 50))
    end
    f.Registry[qJ].Color = function()
        return UJ.WarningBox.IsNormal == true and f.Scheme.OutlineColor or(Color3.fromRGB(169, 0, 0))
    end
end, RefreshSides = function(bJ)
    bJ = H.Visible and g.Size.Y.Offset + 8 or 0
    for H, H in pairs(UJ.Sides) do
        H.Position = UDim2.new(H.Position.X.Scale, 0, 0, bJ)
        H.Size = UDim2.new(0.5, -3, 1, -bJ)
    end
end, Resize = function(H, bJ)
    if bJ then local bJ, JJ, JJ = math.floor(x.AbsoluteSize.Y / 3.25), f:GetTextBounds(WJ.Text, f.Scheme.Font, WJ.TextSize, WJ.AbsoluteSize.X)
    H = 24 + JJ
    if UJ.WarningBox.LockSize == true and H >= bJ then
        NJ.CanvasSize = UDim2.fromOffset(0, H)
        H = bJ
    else
        NJ.CanvasSize = UDim2.fromOffset(0, 0)
    end
    WJ.Size = UDim2.new(1, -4, 0, JJ)
    g.Size = UDim2.new(1, -5, 0, H + 4)
end
UJ:RefreshSides()
end, AddGroupbox = function(H, g)
    H = F("Frame", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), Parent = g.Side == 1 and V or MJ})
    F("UIListLayout", {Padding = UDim.new(0, 6), Parent = H})
    F("UIPadding", {PaddingBottom = UDim.new(0, 4), PaddingTop = UDim.new(0, 4), Parent = H})
    local bJ, JJ, NJ
    do
        bJ = F("Frame", {BackgroundColor3 = "BackgroundColor", Size = UDim2.fromScale(1, 0), Parent = H})
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = bJ})
        f:AddOutline(bJ)
        f:MakeLine(bJ, {Position = UDim2.fromOffset(0, 38), Size = UDim2.new(1, 0, 0, 1)})
        local AJ = f:GetCustomIcon(g.IconName)
        if AJ then
            F("ImageLabel", {Image = AJ.Url, ImageColor3 = AJ.Custom and "WhiteColor" or "AccentColor", ImageRectOffset = AJ.ImageRectOffset, ImageRectSize = AJ.ImageRectSize, Position = UDim2.fromOffset(6, 6), Size = UDim2.fromOffset(22, 22), Parent = bJ})
        end
        local qJ = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(AJ and 24 or 0, 0), Size = UDim2.new(1, 0, 0, 38), Text = g.Name, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left, Parent = bJ})
        F("UIPadding", {PaddingLeft = UDim.new(0, 12), PaddingRight = UDim.new(0, 12), Parent = qJ})
        NJ = F("Frame", {BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 39), Size = UDim2.new(1, 0, 1, -39), Parent = bJ})
        JJ = F("UIListLayout", {Padding = UDim.new(0, 10), Parent = NJ})
        F("UIPadding", {PaddingBottom = UDim.new(0, 10), PaddingLeft = UDim.new(0, 10), PaddingRight = UDim.new(0, 10), PaddingTop = UDim.new(0, 10), Parent = NJ})
    end
    local AJ = {BoxHolder = H, Holder = bJ, Container = NJ, Tab = UJ, DependencyBoxes = {}, Elements = {}, Resize = function(H)
        bJ.Size = UDim2.new(1, 0, 0, JJ.AbsoluteContentSize.Y / f.DPIScale + 59)
    end
    }
    setmetatable(AJ, s)
    AJ:Resize()
    UJ.Groupboxes[g.Name] = AJ
    return AJ
end, AddLeftGroupbox = function(H, H, g)
    return UJ:AddGroupbox({Side = 1, Name = H, IconName = g})
end, AddRightGroupbox = function(H, H, g)
    return UJ:AddGroupbox({Side = 2, Name = H, IconName = g})
end, AddTabbox = function(H, g)
    H = F("Frame", {AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, Size = UDim2.fromScale(1, 0), Parent = g.Side == 1 and V or MJ})
    F("UIListLayout", {Padding = UDim.new(0, 6), Parent = H})
    F("UIPadding", {PaddingBottom = UDim.new(0, 4), PaddingTop = UDim.new(0, 4), Parent = H})
    local V, MJ
    do
        V = F("Frame", {BackgroundColor3 = "BackgroundColor", Size = UDim2.fromScale(1, 0), Parent = H})
        F("UICorner", {CornerRadius = UDim.new(0, W.CornerRadius), Parent = V})
        f:AddOutline(V)
        MJ = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 34), Parent = V})
        F("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, HorizontalFlex = Enum.UIFlexAlignment.Fill, Parent = MJ})
    end
    local bJ
    bJ = {ActiveTab = nil, BoxHolder = H, Holder = V, Tabs = {}, AddTab = function(H, H, JJ)
        local NJ = F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 0, Size = UDim2.fromOffset(0, 34), Text = H, TextSize = 15, TextTransparency = JJ and 0.8 or 0.5, Parent = MJ})
        local MJ, AJ = f:MakeLine(NJ, {AnchorPoint = Vector2.new(0, 1), Position = UDim2.new(0, 0, 1, 1), Size = UDim2.new(1, 0, 0, 1)}), F("Frame", {BackgroundTransparency = 1, Position = UDim2.fromOffset(0, 35), Size = UDim2.new(1, 0, 1, -35), Visible = false, Parent = V})
        local qJ = F("UIListLayout", {Padding = UDim.new(0, 8), Parent = AJ})
        F("UIPadding", {PaddingBottom = UDim.new(0, 7), PaddingLeft = UDim.new(0, 7), PaddingRight = UDim.new(0, 7), PaddingTop = UDim.new(0, 7), Parent = AJ})
        local WJ
        WJ = {ButtonHolder = NJ, Container = AJ, Tab = UJ, Elements = {}, DependencyBoxes = {}, Show = function(LJ)
            if bJ.ActiveTab then bJ.ActiveTab:Hide()
        end
        NJ.BackgroundTransparency = 1
        NJ.TextTransparency = 0
        MJ.Visible = false
        AJ.Visible = true
        bJ.ActiveTab = WJ
        WJ:Resize()
    end, Hide = function(LJ)
        NJ.BackgroundTransparency = 0
        NJ.TextTransparency = 0.5
        MJ.Visible = true
        AJ.Visible = false
        bJ.ActiveTab = nil
    end, Resize = function(MJ)
        if bJ.ActiveTab ~= WJ then return
    end
    V.Size = UDim2.new(1, 0, 0, qJ.AbsoluteContentSize.Y / f.DPIScale + 49)
end
}
if JJ then
    WJ.Disabled = true
end
if not bJ.ActiveTab and not JJ then
    WJ:Show()
end
NJ.MouseButton1Click:Connect(function()
    if WJ.Disabled then return
end
WJ:Show()
end)
setmetatable(WJ, s)
bJ.Tabs[H] = WJ
return WJ
end
}
if g.Name then
    UJ.Tabboxes[g.Name] = bJ
else
    table.insert(UJ.Tabboxes, bJ)
end
return bJ
end, AddLeftTabbox = function(H, H)
    return UJ:AddTabbox({Side = 1, Name = H})
end, AddRightTabbox = function(H, H)
    return UJ:AddTabbox({Side = 2, Name = H})
end, Hover = function(H, H)
    if f.ActiveTab == UJ then return
end
B:Create(e, f.TweenInfo, {TextTransparency = H and 0.25 or 0.5}):Play()
if I then
    B:Create(I, f.TweenInfo, {ImageTransparency = H and 0.25 or 0.5}):Play()
end
end, Show = function(H)
    if f.ActiveTab then f.ActiveTab:Hide()
end
f:PlaySound("click", 0.15, 1)
B:Create(c, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0}):Play()
B:Create(e, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {TextTransparency = 0}):Play()
if I then
    B:Create(I, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {ImageTransparency = 0}):Play()
end
if p then
    o:ShowTabInfo(v, p)
end
x.Position = UDim2.fromOffset(0, 10)
x.Visible = true
B:Create(x, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = UDim2.fromOffset(0, 0)}):Play()
f.ActiveTab = UJ
if f.Searching then
    f:UpdateSearch(f.SearchText)
end
end, Hide = function(H)
    B:Create(c, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {BackgroundTransparency = 1}):Play()
    B:Create(e, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {TextTransparency = 0.5}):Play()
    if I then
        B:Create(I, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {ImageTransparency = 0.5}):Play()
    end
    x.Visible = false
    o:HideTabInfo()
    f.ActiveTab = nil
end
}
if not f.ActiveTab then
    UJ:Show()
end
c.MouseEnter:Connect(function()
    f:PlaySound("hover", 0.08, 1.5)
    UJ:Hover(true)
end)
c.MouseLeave:Connect(function()
    UJ:Hover(false)
end)
c.MouseButton1Click:Connect(UJ.Show)
f.Tabs[v] = UJ
return UJ
end
o.AddKeyTab = function(H, ...)
    local v, p, g
    if select("#", ...) == 1 and typeof(...) == "table" then
        H = select(1, ...)
        v = H.Name or "Tab"
        g, p = H.Icon, H.Description
    else
        v = select(1, ...) or "Tab"
        g, p = select(2, ...), select(3, ...)
    end
    g = g or "key"
    local c, e, I, x
    g = g == "key" and X or(f:GetCustomIcon(g))
    do
        c = F("TextButton", {BackgroundColor3 = "MainColor", BackgroundTransparency = 1, Size = UDim2.new(1, 0, 0, 40), Text = "", Parent = D})
        H = F("UIPadding", {PaddingBottom = UDim.new(0, T and 6 or 11), PaddingLeft = UDim.new(0, T and 6 or 12), PaddingRight = UDim.new(0, T and 6 or 12), PaddingTop = UDim.new(0, T and 6 or 11), Parent = c})
        e = F("TextLabel", {BackgroundTransparency = 1, Position = UDim2.fromOffset(30, 0), Size = UDim2.new(1, -30, 1, 0), Text = v, TextSize = 16, TextTransparency = 0.5, TextXAlignment = Enum.TextXAlignment.Left, Visible = not T, Parent = c})
        if g then
            I = F("ImageLabel", {Image = g.Url, ImageColor3 = g.Custom and "WhiteColor" or "AccentColor", ImageRectOffset = g.ImageRectOffset, ImageRectSize = g.ImageRectSize, ImageTransparency = 0.5, Size = UDim2.fromScale(1, 1), SizeConstraint = T and Enum.SizeConstraint.RelativeXY or Enum.SizeConstraint.RelativeYY, Parent = c})
        end
        table.insert(f.TabButtons, {Label = e, Padding = H, Icon = I})
        x = F("ScrollingFrame", {AutomaticCanvasSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, CanvasSize = UDim2.fromScale(0, 0), ScrollBarThickness = 0, Size = UDim2.fromScale(1, 1), Visible = false, Parent = K})
        F("UIListLayout", {HorizontalAlignment = Enum.HorizontalAlignment.Center, Padding = UDim.new(0, 8), VerticalAlignment = Enum.VerticalAlignment.Center, Parent = x})
        F("UIPadding", {PaddingLeft = UDim.new(0, 1), PaddingRight = UDim.new(0, 1), Parent = x})
    end
    local H
    H = {Elements = {}, IsKeyTab = true, AddKeyBox = function(g, X)
        assert(typeof(X) == "function", "Callback must be a function")
        g = F("Frame", {BackgroundTransparency = 1, Size = UDim2.new(0.75, 0, 0, 21), Parent = x})
        local V = F("TextBox", {BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, PlaceholderText = "Key", Size = UDim2.new(1, -71, 1, 0), TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left, Parent = g})
        F("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), Parent = V})
        local K = F("TextButton", {AnchorPoint = Vector2.new(1, 0), BackgroundColor3 = "MainColor", BorderColor3 = "OutlineColor", BorderSizePixel = 1, Position = UDim2.fromScale(1, 0), Size = UDim2.new(0, 63, 1, 0), Text = "Execute", TextSize = 14, Parent = g})
        K.InputBegan:Connect(function(g)
            if not a(g) then return
        end
        if not f:MouseIsOverFrame(K, g.Position) then
            return
        end
        X(V.Text)
    end)
end, RefreshSides = function(g)
end, Resize = function(g)
end, Hover = function(g, g)
    if f.ActiveTab == H then return
end
B:Create(e, f.TweenInfo, {TextTransparency = g and 0.25 or 0.5}):Play()
if I then
    B:Create(I, f.TweenInfo, {ImageTransparency = g and 0.25 or 0.5}):Play()
end
end, Show = function(g)
    if f.ActiveTab then f.ActiveTab:Hide()
end
B:Create(c, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 0}):Play()
B:Create(e, TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {TextTransparency = 0}):Play()
if I then
    B:Create(I, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {ImageTransparency = 0}):Play()
end
x.Visible = true
if p then
    o:ShowTabInfo(v, p)
end
f.ActiveTab = H
if f.Searching then
    f:UpdateSearch(f.SearchText)
end
end, Hide = function(p)
    B:Create(c, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {BackgroundTransparency = 1}):Play()
    B:Create(e, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {TextTransparency = 0.5}):Play()
    if I then
        B:Create(I, TweenInfo.new(0.15, Enum.EasingStyle.Sine, Enum.EasingDirection.In), {ImageTransparency = 0.5}):Play()
    end
    x.Visible = false
    o:HideTabInfo()
    f.ActiveTab = nil
end
}
if not f.ActiveTab then
    H:Show()
end
c.MouseEnter:Connect(function()
    H:Hover(true)
end)
c.MouseLeave:Connect(function()
    H:Hover(false)
end)
c.MouseButton1Click:Connect(H.Show)
H.Container = x
setmetatable(H, s)
f.Tabs[v] = H
return H
end
function f.Toggle(s, H)
    if typeof(H) == "boolean" then f.Toggled = H
else
    f.Toggled = not f.Toggled
end
if m then
    m.Visible = k
    n = k
end
for v, v in pairs(E) do
    pcall(function()
        v:Cancel()
    end)
end
E = {}
if f.Toggled then
    f:PlaySound("toggle_on", 0.4)
    P.AnchorPoint = Vector2.new(0.5, 0.5)
    P.Size = UDim2.fromOffset(W.Size.X.Offset * 0.92, W.Size.Y.Offset * 0.92)
    P.Visible = true
    s = W.GlassType == "Full" and 0.3 or(W.GlassType == "TopOnly" and 1 or 0)
    P.BackgroundTransparency = 1
    H = B:Create(P, TweenInfo.new(0.15, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = W.Size, BackgroundTransparency = s})
    table.insert(E, H)
    H:Play()
    if W.EnableBlur then
        j.Visible = true
        local s, H = B:Create(j, TweenInfo.new(0.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 0.5}), B:Create(O, TweenInfo.new(0.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = 56})
        table.insert(E, s)
        table.insert(E, H)
        s:Play()
        H:Play()
    end
else
    f:PlaySound("toggle_off", 0.4)
    local s = B:Create(P, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.fromOffset(W.Size.X.Offset * 0.92, W.Size.Y.Offset * 0.92), BackgroundTransparency = 1})
    table.insert(E, s)
    s:Play()
    s.Completed:Connect(function()
        if not f.Toggled then P.Visible = false
        P.AnchorPoint = Vector2.new(0, 0)
    end
end)
local s, H = B:Create(j, TweenInfo.new(0.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {BackgroundTransparency = 1}), B:Create(O, TweenInfo.new(0.2, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {Size = 0})
table.insert(E, s)
table.insert(E, H)
s:Play()
H:Play()
s.Completed:Connect(function()
    if not f.Toggled then j.Visible = false
end
end)
end
if W.UnlockMouseWhileOpen then
    N.Modal = f.Toggled
end
if f.Toggled and not f.IsMobile then
    local N = U.MouseIconEnabled
    pcall(function()
        q:UnbindFromRenderStep("ShowCursor")
    end)
    q:BindToRenderStep("ShowCursor", Enum.RenderPriority.Last.Value, function()
        U.MouseIconEnabled = not f.ShowCustomCursor
        d.Position = UDim2.fromOffset(C.X, C.Y)
        d.Visible = f.ShowCustomCursor
        if not(f.Toggled and b and b.Parent) then
            U.MouseIconEnabled = N
            d.Visible = false
            q:UnbindFromRenderStep("ShowCursor")
        end
    end)
elseif not f.Toggled then
    Q.Visible = false
    for N, N in pairs(f.Options) do
        if N.Type == "ColorPicker" then
            N.ColorMenu:Close()
            N.ContextMenu:Close()
        elseif N.Type == "Dropdown" or N.Type == "KeyPicker" then
            N.Menu:Close()
        end
    end
end
end
if W.EnableSidebarResize then
    local N, s, C = (W.MinSidebarWidth + W.SidebarCompactWidth) * W.SidebarCollapseThreshold
    local H, Q = false
    i = F("TextButton", {AnchorPoint = Vector2.new(0.5, 0), BackgroundTransparency = 1, Position = UDim2.fromScale(0.5, 0), Size = UDim2.new(0, 8, 1, 0), Text = "", Parent = Y})
    i.MouseEnter:Connect(function()
        B:Create(Y, f.TweenInfo, {BackgroundColor3 = f:GetLighterColor(f.Scheme.OutlineColor)}):Play()
    end)
    i.MouseLeave:Connect(function()
        if H then return
    end
    B:Create(Y, f.TweenInfo, {BackgroundColor3 = f.Scheme.OutlineColor}):Play()
end)
i.InputBegan:Connect(function(v)
    if not a(v) then return
end
f.CantDragForced = true
s = v.Position
C, H = o:GetSidebarWidth(), true
Q = v.Changed:Connect(function()
    if v.UserInputState ~= Enum.UserInputState.End then return
end
f.CantDragForced = false
B:Create(Y, f.TweenInfo, {BackgroundColor3 = f.Scheme.OutlineColor}):Play()
H = false
if Q and Q.Connected then
    Q:Disconnect()
    Q = nil
end
end)
end)
f:GiveSignal(U.InputChanged:Connect(function(v)
    if not f.Toggled or not(b and b.Parent) then H = false
    if Q and Q.Connected then
        Q:Disconnect()
        Q = nil
    end
    return
end
if H and(t(v)) then
    local H = v.Position -s
    local s = C + H.X
    if W.DisableCompactingSnap then
        o:SetSidebarWidth(s)
        return
    end
    if s > N then
        o:SetSidebarWidth(math.max(s, W.MinSidebarWidth))
    else
        o:SetSidebarWidth(W.SidebarCompactWidth)
    end
end
end))
end
if W.EnableCompacting and W.SidebarCompacted then
    o:SetSidebarWidth(W.SidebarCompactWidth)
end
if W.AutoShow then
    task.spawn(f.Toggle)
end
if f.IsMobile then
    J, L = f:AddDraggableButton("Toggle", function()
        f:Toggle()
    end, true), f:AddDraggableButton("Lock", function(N)
        f.CantDragForced = not f.CantDragForced
        N:SetText(f.CantDragForced and "Unlock" or "Lock")
    end, true)
    o._mobileButtons = {J, L}
    if W.MobileButtonsSide == "Right" then
        J.Button.Position = UDim2.new(1, -6, 0, 6)
        J.Button.AnchorPoint = Vector2.new(1, 0)
        L.Button.Position = UDim2.new(1, -6, 0, 46)
        L.Button.AnchorPoint = Vector2.new(1, 0)
    else
        L.Button.Position = UDim2.fromOffset(6, 46)
    end
end
_:GetPropertyChangedSignal("Text"):Connect(function()
    f:UpdateSearch(_.Text)
end)
f:GiveSignal(U.InputBegan:Connect(function(J)
    if f.Unloaded then return
end
if U:GetFocusedTextBox() then
    return
end
if typeof(f.ToggleKeybind) == "table" and f.ToggleKeybind.Type == "KeyPicker" and J.KeyCode.Name == f.ToggleKeybind.Value or J.KeyCode == f.ToggleKeybind then
    f.Toggle()
end
end))
f:GiveSignal(U.WindowFocused:Connect(function()
    f.IsRobloxFocused = true
end))
f:GiveSignal(U.WindowFocusReleased:Connect(function()
    f.IsRobloxFocused = false
end))
o.SetSnowfall = function(J, N)
    k = N
    if N and not m then
        m, n = F("Frame", {BackgroundTransparency = 1, Size = UDim2.fromScale(1, 1), ZIndex = 10, Visible = true, Parent = b}), true
        local L, s = game:GetService("RunService"), {}
        for C = 1, 100, 1 do
            local H, Q, v, p = math.random(3, 6), math.random(30, 60) / 100, math.random(0, 100) / 100, math.random(- 50, -5) / 100
            J = F("Frame", {BackgroundColor3 = Color3.fromRGB(255, 255, 255), BackgroundTransparency = Q, Size = UDim2.fromOffset(H, H), Position = UDim2.fromScale(v, p), Rotation = math.random(0, 360), Parent = m})
            F("UICorner", {CornerRadius = UDim.new(1, 0), Parent = J})
            s[C] = {frame = J, velY = 0.012, velX = math.random(- 3, 3) / 1000, rotSpeed = math.random(- 60, 60) / 100, delay = math.random(0, 150) / 100, elapsed = 0, posY = p, posX = v, rot = math.random(0, 360)}
        end
        f:GiveSignal((L.Heartbeat:Connect(function(J)
            if not n then return
        end
        local L = J * 60
        for C, C in pairs(s) do
            if C.elapsed < C.delay then
                C.elapsed = C.elapsed + J
            else
                C.posY = C.posY + C.velY * G * L
                C.posX = C.posX + C.velX * L
                C.rot = C.rot + C.rotSpeed * L
                if C.posY > 1.2 then
                    C.posY = math.random(- 30, -5) / 100
                    C.posX = math.random(0, 100) / 100
                end
                if C.posX < -0.05 then
                    C.posX = 1.05
                end
                if C.posX > 1.05 then
                    C.posX = -0.05
                end
                C.frame.Position = UDim2.fromScale(C.posX, C.posY)
                C.frame.Rotation = C.rot
            end
        end
    end)))
elseif not N and m then
    m.Visible = false
    n = false
elseif m then
    m.Visible = N
    n = N
end
end
o.SetBlur = function(J, J)
    W.EnableBlur = J
    if not f.Toggled then
        return
    end
    for N, N in pairs(E) do
        pcall(function()
            N:Cancel()
        end)
    end
    E = {}
    if J then
        j.Visible = true
        local J, N = B:Create(j, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 0.5}), B:Create(O, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = 56})
        table.insert(E, J)
        table.insert(E, N)
        J:Play()
        N:Play()
    else
        local J, N = B:Create(j, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1}), B:Create(O, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = 0})
        table.insert(E, J)
        table.insert(E, N)
        J:Play()
        N:Play()
        J.Completed:Connect(function()
            j.Visible = false
        end)
    end
end
o.SetGlassType = function(J, N)
    W.GlassType = N
    if S then
        S:Destroy()
        S = nil
    end
    J = P:FindFirstChild("TopBar")
    if J then
        J.BackgroundTransparency = 1
    end
    if N == "Full" then
        P.BackgroundTransparency = 0.3
        for L, L in pairs(y) do
            if L and L.Parent then
                L.BackgroundTransparency = 1
            end
        end
        S = F("UIStroke", {Color = Color3.fromRGB(255, 255, 255), Transparency = 0.9, Thickness = 1, Parent = P})
    elseif N == "TopOnly" then
        for N, N in pairs(y) do
            if N and N.Parent then
                N.BackgroundTransparency = 0
            end
        end
        J = P:FindFirstChild("TopBar")
        if J then
            P.BackgroundTransparency = 1
            J.BackgroundTransparency = 0.3
            S = F("UIStroke", {Color = Color3.fromRGB(255, 255, 255), Transparency = 0.9, Thickness = 1, Parent = J})
        end
    else
        P.BackgroundTransparency = 0
        for J, J in pairs(y) do
            if J and J.Parent then
                J.BackgroundTransparency = 0
            end
        end
    end
end
o.SetCustomBackground = function(J, N)
    J = f._customBg
    if not J then
        return
    end
    if not N or N == "" then
        J.Image = ""
        J.Visible = false
        return
    end
    local L = tostring(N)
    J.Image = if not L:match("://") then "rbxassetid://" .. L:gsub("%D", "") else L
    J.Visible = true
end
o.SetBackgroundOpacity = function(J, N)
    J = f._customBg
    if not J then
        return
    end
    J.ImageTransparency = 1 -math.clamp(N, 0, 1)
end
o.SetMobileButtonOpacity = function(J, J)
    local N = 1 -math.clamp(J, 0, 1)
    for J, J in pairs(o._mobileButtons or{}) do
        if J and J.Button then
            pcall(function()
                J.Button.BackgroundTransparency = N
                J.Button.TextTransparency = N
                for L, L in ipairs(J.Button:GetChildren()) do
                    if L:IsA("UIStroke") then
                        L.Transparency = N
                    end
                end
            end)
        end
    end
end
o.SetSnowColor = function(J, J)
    if not m then return
end
for N, N in m:GetChildren() do
    if N:IsA("Frame") then
        N.BackgroundColor3 = J
    end
end
end
o.SetSnowSpeed = function(J, J)
    G = J or 1
end
o.SetSnowAmount = function(J, N)
    if not m then return
end
J = #m:GetChildren()
if N > J then
    for L = J + 1, N, 1 do
        local L = F("Frame", {BackgroundColor3 = Color3.fromRGB(255, 255, 255), BackgroundTransparency = math.random(20, 60) / 100, Size = UDim2.fromOffset(math.random(2, 6), math.random(2, 6)), Position = UDim2.fromScale(math.random(0, 100) / 100, -0.1), Parent = m})
        F("UICorner", {CornerRadius = UDim.new(1, 0), Parent = L})
        task.spawn(function()
            while n and L and L.Parent do
                local s, C, H = math.random(8, 15) / (G or 1), math.random(110, 130) / 100, L.Position.X.Scale + math.random(- 20, 20) / 100
                B:Create(L, TweenInfo.new(s, Enum.EasingStyle.Linear), {Position = UDim2.fromScale(H, C)}):Play()
                task.wait(s)
                if n then
                    L.Position = UDim2.fromScale(math.random(0, 100) / 100, -0.1)
                end
            end
        end)
    end
elseif N < J then
    for L = J, N + 1, -1 do
        local J = m:GetChildren()[L]
        if J then
            J:Destroy()
        end
    end
end
end
o.EnableSidebarHover = function(J, J)
    if f.IsMobile then return
end
J = J or{}
local N, L, B = J.CollapsedWidth or 54, J.ExpandedWidth or(o:GetSidebarWidth()), J.Speed or 18
W.EnableCompacting = false
local s, C, H = L, o:GetSidebarWidth(), -1
local function Q(v)
    local p, g = N / 2, math.clamp((v -0.12) / 0.6, 0, 1)
    g = g * g * (3 -2 * g)
    for v, _ in pairs(f.TabButtons) do
        if _.Padding then
            _.Padding.PaddingLeft = UDim.new(0, 0)
            _.Padding.PaddingRight = UDim.new(0, 0)
            _.Padding.PaddingTop = UDim.new(0, 0)
            _.Padding.PaddingBottom = UDim.new(0, 0)
        end
        if _.Icon then
            _.Icon.AnchorPoint = Vector2.new(0.5, 0.5)
            _.Icon.SizeConstraint = Enum.SizeConstraint.RelativeYY
            _.Icon.Size = UDim2.fromScale(0.46, 0.46)
            _.Icon.Position = UDim2.new(0, p, 0.5, 0)
        end
        if _.Label then
            _.Label.Visible = true
            _.Label.Position = UDim2.fromOffset(p + 14 + 8 * g, 0)
            v = _.Label.Parent.BackgroundTransparency < 0.5 and 0 or 0.5
            _.Label.TextTransparency = v + (1 -v) * (1 -g)
        end
    end
    Z.Visible = true
    Z.TextTransparency = 1 -g
end
f:GiveSignal(q.RenderStepped:Connect(function(v)
    if not(b and b.Parent and P.Visible) then return
end
C = if math.abs(s -C) >= 0.5 then C + (s -C) * (1 -math.exp(- v * B)) else s
if math.abs(C -H) >= 0.1 then
    H = C
    o:SetSidebarWidth(C)
end
Q(math.clamp((C -N) / math.max(1, L -N), 0, 1))
end))
local B = 0
local function H()
    B += 1
    s = L
end
local function L()
    B = math.max(0, B -1)
    task.delay(0.06, function()
        local v = B
        if v <= 0 then
            s = N
        end
    end)
end
J = {D, l, Y}
for B, B in ipairs(J) do
    f:GiveSignal(B.MouseEnter:Connect(H))
    f:GiveSignal(B.MouseLeave:Connect(L))
end
s, C = N, N
o:SetSidebarWidth(N)
Q(0)
end
o.EnableResize = function(J, J)
    J = J or{}
    local N, L, B, s = J.Thickness or 6, J.CornerSize or 12, J.Smooth ~= false, Color3.fromRGB(6, 6, 8)
    local J, C, H = F("ImageLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, ImageColor3 = s, ImageTransparency = 0.35, Position = UDim2.new(0.5, 2, 0.5, 3), Size = UDim2.fromOffset(21, 21), ZIndex = 997, Visible = false, Parent = d}), {Vector2.new(1, 0), Vector2.new(- 1, 0), Vector2.new(0, 1), Vector2.new(0, -1), Vector2.new(1, 1), Vector2.new(1, -1), Vector2.new(- 1, 1), Vector2.new(- 1, -1)}, {}
    for Y, Y in ipairs(C) do
        H[#H + 1] = F("ImageLabel", {AnchorPoint = Vector2.new(0.5, 0.5), BackgroundTransparency = 1, ImageColor3 = s, Position = UDim2.new(0.5, Y.X * 1.6, 0.5, Y.Y * 1.6), Size = UDim2.fromOffset(20, 20), ZIndex = 999, Visible = false, Parent = d})
    end
    local function s(C, Y)
        C.Image = Y.Url
        C.ImageRectOffset = Y.ImageRectOffset
        C.ImageRectSize = Y.ImageRectSize
        C.Visible = true
    end
    local function C(Y)
        local l = Y and(f:GetIcon(Y))
        if not l then
            return
        end
        d.BackgroundTransparency = 1
        for Y, Y in ipairs(d:GetChildren()) do
            if Y:IsA("Frame") then
                Y.Visible = false
            end
        end
        s(w, l)
        s(J, l)
        for Y, Y in ipairs(H) do
            s(Y, l)
        end
    end
    local function s()
        for Y, Y in ipairs(d:GetChildren()) do
            if Y:IsA("Frame") then
                Y.Visible = true
            end
        end
        d.BackgroundTransparency = 0
        w.Visible = false
        J.Visible = false
        for J, J in ipairs(H) do
            J.Visible = false
        end
    end
    local J, d, w = false, P:FindFirstChild("TopBar"), false
    if d then
        f:GiveSignal(d.InputBegan:Connect(function(d)
            if a(d) and not f.CantDragForced then w, J = true, true
            C("move")
        end
    end))
end
f:GiveSignal(U.InputEnded:Connect(function(d)
    if w and d.UserInputType == Enum.UserInputType.MouseButton1 then w, J = false, false
    s()
end
end))
local d, w, H, Y, l = P.Size.X.Offset, P.Size.Y.Offset, P.Position.X.Offset, P.Position.Y.Offset, false
if B then
    f:GiveSignal(q.RenderStepped:Connect(function(q)
        if not l or not(b and b.Parent and P.Visible) then return
    end
    local Q, v, Z = P.Size, P.Position, math.clamp(q * 16, 0, 1)
    P.Size = UDim2.new(Q.X.Scale, Q.X.Offset + (d -Q.X.Offset) * Z, Q.Y.Scale, Q.Y.Offset + (w -Q.Y.Offset) * Z)
    P.Position = UDim2.new(v.X.Scale, v.X.Offset + (H -v.X.Offset) * Z, v.Y.Scale, v.Y.Offset + (Y -v.Y.Offset) * Z)
end))
end
local function q(Q, v, Z, p)
    local g = F("TextButton", {Active = true, AutoButtonColor = false, BackgroundTransparency = 1, Text = "", ZIndex = 60, Parent = P})
    p(g)
    local p, F, _, c, e, I = false
    g.MouseEnter:Connect(function()
        if not J then C(Z)
    end
end)
g.MouseLeave:Connect(function()
    if not J then s()
end
end)
g.InputBegan:Connect(function(g)
    if not a(g) then return
end
p, l, J = true, true, true
C(Z)
F = g.Position
d, w = P.Size.X.Offset, P.Size.Y.Offset
H, Y, _, c = P.Position.X.Offset, P.Position.Y.Offset, d, w
e, I = H, Y
end)
f:GiveSignal(U.InputEnded:Connect(function(C)
    if p and C.UserInputType == Enum.UserInputType.MouseButton1 then p, l, J = false, false, false
    s()
    W.Size = UDim2.fromOffset(d, w)
    f._persistentWindowSize = W.Size
    for J, J in pairs(f.Tabs) do
        pcall(function()
            J:Resize(true)
        end)
    end
end
end))
f:GiveSignal(U.InputChanged:Connect(function(M)
    if not p or not t(M) then return
end
if not(b and b.Parent) then
    p, l = false, false
    return
end
local b, J = workspace.CurrentCamera.ViewportSize, M.Position -F
local M, W = math.clamp(_ + Q * J.X, f.MinSize.X, b.X -20), math.clamp(c + v * J.Y, f.MinSize.Y, b.Y -20)
H = e + Q * (M -_) / 2
Y, d, w = I + v * (W -c) / 2, M, W
if not B then
    P.Size = UDim2.new(P.Size.X.Scale, d, P.Size.Y.Scale, w)
    P.Position = UDim2.new(P.Position.X.Scale, H, P.Position.Y.Scale, Y)
end
end))
end
q(0, -1, "move-vertical", function(M)
    M.AnchorPoint = Vector2.new(0.5, 0)
    M.Position = UDim2.new(0.5, 0, 0, 0)
    M.Size = UDim2.new(1, -L * 2, 0, N)
end)
q(0, 1, "move-vertical", function(M)
    M.AnchorPoint = Vector2.new(0.5, 1)
    M.Position = UDim2.new(0.5, 0, 1, 0)
    M.Size = UDim2.new(1, -L * 2, 0, N)
end)
q(- 1, 0, "move-horizontal", function(M)
    M.AnchorPoint = Vector2.new(0, 0.5)
    M.Position = UDim2.new(0, 0, 0.5, 0)
    M.Size = UDim2.new(0, N, 1, -L * 2)
end)
q(1, 0, "move-horizontal", function(M)
    M.AnchorPoint = Vector2.new(1, 0.5)
    M.Position = UDim2.new(1, 0, 0.5, 0)
    M.Size = UDim2.new(0, N, 1, -L * 2)
end)
q(- 1, -1, "move-diagonal", function(M)
    M.AnchorPoint = Vector2.new(0, 0)
    M.Position = UDim2.new(0, 0, 0, 0)
    M.Size = UDim2.fromOffset(L, L)
end)
q(1, 1, "move-diagonal", function(M)
    M.AnchorPoint = Vector2.new(1, 1)
    M.Position = UDim2.new(1, 0, 1, 0)
    M.Size = UDim2.fromOffset(L, L)
end)
q(1, -1, "move-diagonal-2", function(M)
    M.AnchorPoint = Vector2.new(1, 0)
    M.Position = UDim2.new(1, 0, 0, 0)
    M.Size = UDim2.fromOffset(L, L)
end)
q(- 1, 1, "move-diagonal-2", function(M)
    M.AnchorPoint = Vector2.new(0, 1)
    M.Position = UDim2.new(0, 0, 1, 0)
    M.Size = UDim2.fromOffset(L, L)
end)
end
return o
end
local function M()
    if f.Unloaded then return
end
local b, J = r(), r(true)
for N, N in pairs(z) do
    if N.Type == "Dropdown" and N.SpecialType == "Player" then
        N:SetValues(N.ExcludeLocalPlayer and J or b)
    end
end
end
local function b()
    if f.Unloaded then return
end
local J = u()
for N, N in pairs(z) do
    if N.Type == "Dropdown" and N.SpecialType == "Team" then
        N:SetValues(J)
    end
end
end
f:GiveSignal(A.PlayerAdded:Connect(M))
f:GiveSignal(A.PlayerRemoving:Connect(M))
f:GiveSignal(h.ChildAdded:Connect(b))
f:GiveSignal(h.ChildRemoved:Connect(b))
R().YabujinLibrary = f
return f
end
end

-- ============================================================================
-- PART 4: Luraph v14.x Crypto Primitives (ChaCha20 & RC4) & Decoy Functions
-- ============================================================================

-- Function [54]: chacha20CoreRounds (550 bytes) — Luraph ChaCha20 stream cipher quarter-round block mixer (modulo 2^32, rotations 16, 12, 8, 7).
YabujinDeobfuscated.chacha20CoreRounds = function(M, b)
    return function(J, N)
        local A = J[13]
        for A = 1, N, 2 do
            for N = 1, 32, 4 do
                local q, W, U, L = M:Yq(b[1], N), M:Yq(b[1], N + 1), M:Yq(b[1], N + 2), M:Yq(b[1], N + 3)
                local b, N, h, B = J[q], J[W], J[U], J[L]
                b = (b + N) % 4294967296
                A = upval_30(B, b)
                B = upval_123(upval_33(A, 16), (upval_74(A, 16))) % 4294967296
                h = (h + B) % 4294967296
                A = upval_30(N, h)
                N = upval_123(upval_33(A, 12), (upval_74(A, 20))) % 4294967296
                b = upval_21(b + N, 4294967295)
                A = upval_30(B, b)
                B = upval_123(upval_33(A, 8), (upval_74(A, 24))) % 4294967296
                h = (h + B) % 4294967296
                A = upval_30(N, h)
                N = upval_123(upval_33(A, 7), (upval_74(A, 25))) % 4294967296
                J[q] = b
                J[W] = N
                J[U] = h
                J[L] = B
            end
        end
    end
end

-- Function [115]: rc4BufferDecryptor (1519 bytes) — Luraph RC4 stream cipher buffer decryptor.
YabujinDeobfuscated.rc4BufferDecryptor = function(M, b)
    return function()
        local J, N, A, q, W, U, L, h, B, R, s, d, w = 8
        while true do
            if J <= 4 then
                if J <= 1 then
                    if J <= 0 then
                        local u, C = M[33](U, 16), M[21](q + 1, 255)
                        local o = M[21](W + R[C], 255)
                        local H, z = R[o], R[C]
                        R[C] = H
                        R[o] = z
                        z = M[123](A, u, (M[33](R[M[21](R[C] + R[o], 255)], 24)))
                        M[86](s, w, (M[30](M[19](L, h + w), z)))
                        J, A, q, W, U = 7, C, o, C, o
                    else
                        return s
                    end
                elseif J <= 2 then
                    local u = M[21](W + 1, 255)
                    local C = M[21](U + R[u], 255)
                    R[u], R[C] = R[C], R[u]
                    local o = R[M[21](R[u] + R[C], 255)]
                    M[90](s, A, (M[30](M[70](L, h + A), o)))
                    J, W, U = 9, u, C
                elseif J <= 3 then
                    local u, C = B -1, 1 + 0
                    J, N = 9, {d -C, C, u + 0, nil, N}
                else
                    J, N = 3, N[2]
                end
            elseif J <= 7 then
                if J <= 5 then
                    local u = M[21](W + 1, 255)
                    local C = M[21](U + R[u], 255)
                    R[u], R[C] = R[C], R[u]
                    local o, H = M[123](0, (M[33](R[M[21](R[u] + R[C], 255)], 0))), M[21](u + 1, 255)
                    u = M[21](C + R[H], 255)
                    J, A, q, W, U = 6, o, H, u, R[u]
                elseif J <= 6 then
                    R[q], R[W] = U, R[q]
                    local u, C = M[123](A, (M[33](R[M[21](R[q] + R[W], 255)], 8))), M[21](q + 1, 255)
                    local o = M[21](W + R[C], 255)
                    R[C], R[o] = R[o], R[C]
                    J, A, q, W, U = 0, u, C, o, R[M[21](R[C] + R[o], 255)]
                else
                    local q, u, C = N[3], N[4], N[5]
                    local o, H = q + u, u <= 0
                    local u, z, P = not H, o >= C, o <= C
                    q = H and z or u and P
                    N[3] = o
                    if q then
                        J, w = 5, o
                    else
                        J = 4
                    end
                end
            elseif J <= 8 then
                local q = upval_1
                local w = upval_2
                local u = M[121](q) -w
                local C, o, H = upval_3, M[104](u), u -u % 4
                local M, b = H -1, 4 + 0
                J, N, W, U, L, h, B, R, s, d = 7, {nil, N, 0 -b, b, M + 0}, 0, 0, q, w, u, C, o, H
            elseif J <= 9 then
                local M, b, q = N[1], N[2], N[3]
                local W, U = M + b, b <= 0
                local M, L, h = not U, W >= q, W <= q
                b = U and L or M and h
                N[1] = W
                if b then
                    J, A = 2, W
                else
                    J = 10
                end
            else
                J, N = 1, N[5]
            end
        end
    end
end

-- Function [29]: lcgPrngStepShort (224 bytes) — Luraph LCG PRNG state update helper (modulo 2^28).
YabujinDeobfuscated.lcgPrngStepShort = function(M, b)
    return function()
        local b = 1
        while true do
            if b <= 0 then
                return
            else
                prngState = (108001 * prngState + 41316589) % 268435456
                prngState, b = (277145 * prngState + 18527935) % 268435456, 0
            end
        end
    end
end

-- Function [100]: lcgPrngStateMixer (1835 bytes) — Luraph multi-round LCG PRNG state mixer (modulo 2^28).
YabujinDeobfuscated.lcgPrngStateMixer = function(M, b)
    return function()
        local b, J, N = 3
        while true do
            if b <= 3 then
                if b <= 1 then
                    if b <= 0 then
                        prngState = (J + N) % 268435456
                        prngState = (690929 * prngState + 119601043) % 268435456
                        prngState = (859391 * prngState + 120116449) % 268435456
                        prngState, b = (171135 * prngState + 144943331) % 268435456, 1
                    else
                        prngState = (582423 * prngState + 54050079) % 268435456
                        prngState = (297915 * prngState + 21926831) % 268435456
                        prngState = (443227 * prngState + 143916285) % 268435456
                        b, J, N = 5, 351993 * prngState, 20986297
                    end
                elseif b <= 2 then
                    return
                else
                    prngState = (40009 * prngState + 130141905) % 268435456
                    prngState = (793801 * prngState + 99058595) % 268435456
                    prngState = (164495 * prngState + 14226451) % 268435456
                    b, J = 7, 918999 * prngState + 33991111
                end
            elseif b <= 5 then
                if b <= 4 then
                    prngState = J
                    prngState = (257053 * prngState + 49005783) % 268435456
                    prngState = (878903 * prngState + 15147739) % 268435456
                    prngState = (673811 * prngState + 67595405) % 268435456
                    b, J, N = 0, 650741 * prngState, 222415865
                else
                    prngState = (J + N) % 268435456
                    prngState = (106839 * prngState + 131364431) % 268435456
                    prngState = (677009 * prngState + 59323863) % 268435456
                    prngState, b = (654607 * prngState + 14723519) % 268435456, 2
                end
            elseif b <= 6 then
                prngState = (J * N + 71663543) % 268435456
                prngState = (180075 * prngState + 149212815) % 268435456
                prngState = (310757 * prngState + 149644699) % 268435456
                b, J = 4, (434999 * prngState + 14898349) % 268435456
            else
                prngState = J % 268435456
                prngState = (910865 * prngState + 89397363) % 268435456
                prngState = (635211 * prngState + 129853869) % 268435456
                prngState = (195469 * prngState + 103376131) % 268435456
                b, J, N = 6, 429183, prngState
            end
        end
    end
end

-- Function [57]: luraphDecoyGuiScanner (730 bytes) — Luraph polymorphic chaff/decoy coroutine loop.
YabujinDeobfuscated.luraphDecoyGuiScanner = function(M, b)
    return function()
        while true do
            if not decoyTable1[24](function()
                local b = decoyTable1[1][decoyTable1[2]][decoyTable1[3]][decoyTable1[4]][decoyTable1[5]][decoyTable1[6]][decoyTable1[7]]
                local J = b[decoyTable1[8]](b)
                for N, N in decoyTable1[9](J) do
                    if N[decoyTable1[10]](N, decoyTable1[11]) and(N[decoyTable1[12]](N, decoyTable1[20])) then
                        b = N[decoyTable1[20]][decoyTable1[13]]
                        if decoyTable1[16](decoyTable2, b[decoyTable1[14]](b, 3)) then
                            decoyTarget = N
                            break
                        end
                    end
                end
                if decoyTarget then
                    local b = decoyTarget[decoyTable1[20]][decoyTable1[13]]
                    decoyTarget[decoyTable1[20]][decoyTable1[13]] = b[decoyTable1[14]](b, 1, 2) .. decoyTable2[1][decoyTable1[14]](decoyTable2[1], 1, 10) .. decoySep .. decoyVal1 .. decoyVal2
                    decoyTarget[decoyTable1[17]][decoyTable1[18]] = decoyVal3
                    decoyTarget[decoyTable1[20]][decoyTable1[19]] = decoyVal4
                end
            end) then
                decoyTable1[21](decoyTable1[22])
            else
                decoyTable1[21](decoyTable1[23])
            end
        end
    end
end

-- Function [68]: luraphDecoyProgressBar (614 bytes) — Luraph chaff/decoy progress bar loop.
YabujinDeobfuscated.luraphDecoyProgressBar = function(M, b)
    return function()
        while true do
            if activeFlag then
                local b = progressValue
                local J = progressMax
                if b < J then
                    progressValue += 1
                end
                J = mathFloor(progressValue / 5)
                outputText = strTable[strDecrypt("P", 30780822482938)] .. strTable[strDecrypt("\206", 10650770187967)]:rep(J) .. strTable[strDecrypt("\218", 11767495764687)]:rep(20 -J) .. strTable[strDecrypt("\18 \11", 17444542037878)] .. progressValue .. strTable[strDecrypt(" \207\135\147I", 4720313110828)]
            end
            waitFunc(0.03)
            if stopFlag then
                break
            end
        end
    end
end

return YabujinDeobfuscated
