local BS = { }
	
	BS.TerrorBladeOptionEnabled 	= Menu.AddOptionBool( { "BiT scripts", "Heroes", "TerrorBlade" }, "Enabled", false )
	BS.TerrorBladeComboKey 			= Menu.AddKeyOption( { "BiT scripts", "Heroes", "TerrorBlade" }, "Combo key", Enum.ButtonCode.KEY_SPACE)
	BS.TerrorBladeTargetSelect		= Menu.AddOptionCombo( { "BiT scripts", "Heroes", "TerrorBlade" }, "Target selection", { 'Nearest to mouse', 'Nearest to hero' }, 0 )
	BS.TerrorBladeTargetRadius		= Menu.AddOptionSlider( { "BiT scripts", "Heroes", "TerrorBlade" }, "Target radius", 100, 700, 100)
	BS.TerrorBladeAutoSunderOpt		= Menu.AddOptionBool( { "BiT scripts", "Heroes", "TerrorBlade" }, "Auto sunder", false )
	BS.TerrorBladeSunderThreshold	= Menu.AddOptionSlider( { "BiT scripts", "Heroes", "TerrorBlade" }, "Sunder threshold", 10, 40, 10)
	
	BS.Hero 			= nil
	BS.HeroName			= nil

	BS.Target 			= nil

	BS.Q 				= nil
	BS.W 				= nil
	BS.E 				= nil
	BS.R 				= nil

	BS.D 				= nil
	BS.F 				= nil

	BS.GT 				= 0
	BS.IT 				= 0
	BS.CT 				= 0

	function BS.OnGameStart( ... )
		BS.Hero = Heroes.GetLocal( )

		BS.HeroName = NPC.GetUnitName( BS.Hero )
		BS.Q = NPC.GetAbilityByIndex( BS.Hero, 0 )
		BS.W = NPC.GetAbilityByIndex( BS.Hero, 1 )
		BS.E = NPC.GetAbilityByIndex( BS.Hero, 2 )
		BS.R = NPC.GetAbilityByIndex( BS.Hero, 5 )

		BS.D = NPC.GetAbilityByIndex( BS.Hero, 3 )
		BS.F = NPC.GetAbilityByIndex( BS.Hero, 4 )

		for i = 0, 15 do
			local item = NPC.GetItemByIndex( BS.Hero, i)
			if item and Entity.IsAbility( item ) then
				Log.Write(Ability.GetName(item))
			end
		end
	end

	function BS.OnUpdate( ... )
		if not BS.Hero then BS.OnGameStart( ) return end
		BS.GT = GameRules.GetGameTime( )
		BS.OnUpdateItems( )
		BS.OnUpdateTerrorBlade( )
	end
	function BS.OnUpdateItems( ... )
		if BS.IT > BS.GT then return end
		BS.BM 			= NPC.GetItem( BS.Hero, 'item_blade_mail')
		BS.Mom 			= NPC.GetItem( BS.Hero, 'item_mask_of_madness')
		BS.Bkb 			= NPC.GetItem( BS.Hero, 'item_black_king_bar')
		BS.Hex 			= NPC.GetItem( BS.Hero, 'item_sheepstick')
		BS.Drum 		= NPC.GetItem( BS.Hero, 'item_ancient_janggo')
		BS.Pipe 		= NPC.GetItem( BS.Hero, 'item_hood_of_defiance') or NPC.GetItem( BS.Hero, 'item_pipe')
		BS.Atos 		= NPC.GetItem( BS.Hero, 'item_rod_of_atos')
		BS.Veil 		= NPC.GetItem( BS.Hero, 'item_veil_of_discord' )
		BS.Lotus 		= NPC.GetItem( BS.Hero, 'item_lotus_orb')
		BS.Manta 		= NPC.GetItem( BS.Hero, 'item_manta')
		BS.Necro 		= NPC.GetItem( BS.Hero, 'item_necronomicon') or NPC.GetItem( BS.Hero, 'item_necronomicon_2' ) or NPC.GetItem( BS.Hero, 'item_necronomicon_3' )
		BS.Solar 		= NPC.GetItem( BS.Hero, 'item_solar_crest') or NPC.GetItem( BS.Hero, 'item_medallion_of_courage' )
		BS.Nullf 		= NPC.GetItem( BS.Hero, 'item_nullifier')
		BS.Dagon 		= NPC.GetItem( BS.Hero, 'item_dagon') or NPC.GetItem( BS.Hero, 'item_dagon_2') or NPC.GetItem( BS.Hero, 'item_dagon_3') or NPC.GetItem( BS.Hero, 'item_dagon_4') or NPC.GetItem( BS.Hero, 'item_dagon_5')
		BS.Shiva 		= NPC.GetItem( BS.Hero, 'item_shivas_guard')
		BS.Orchid 		= NPC.GetItem( BS.Hero, 'item_orchid') or NPC.GetItem( BS.Hero, 'item_bloodthorn' )
		BS.Crimson 		= NPC.GetItem( BS.Hero, 'item_crimson_guard')
		BS.Halberd 		= NPC.GetItem( BS.Hero, 'item_heavens_halberd')
		BS.Abyssal		= NPC.GetItem( BS.Hero, 'item_abyssal_blade' )
		BS.Etherial 	= NPC.GetItem( BS.Hero, 'item_ethereal_blade')
		BS.Diffusal 	= NPC.GetItem( BS.Hero, 'item_diffusal_blade')
		BS.IT = BS.GT + 0.1
	end
	function BS.LockTarget( enemy )
		if BS.Target == nil and enemy then
			BS.Target = enemy
			return true
		end
		if BS.Target and Entity.IsHero( BS.Target ) then
			if not Entity.IsAlive( BS.Target ) then
				BS.Target = nil
				return true
			elseif Entity.IsDormant( BS.Target ) then
				BS.Target = nil
				return true
			end
		end
		return false
	end

	function BS.OnUpdateTerrorBlade( ... )
		if not Menu.IsEnabled( BS.TerrorBladeOptionEnabled ) then return end
		if Menu.IsEnabled( BS.TerrorBladeAutoSunderOpt ) then BS.TerrorBladeAutoSunder( ) end
		if Menu.IsKeyDown( BS.TerrorBladeComboKey ) then BS.TerrorBladeCombo( ) return else BS.Target = nil end
	end
	function BS.TerrorBladeCombo( ... )
		if BS.CT > BS.GT then return end
		local enemy = nil
		if not BS.Target then
			if Menu.GetValue( BS.TerrorBladeTargetSelect ) == 0 then
				enemy = Input.GetNearestHeroToCursor( Entity.GetTeamNum( BS.Hero ), Enum.TeamType.TEAM_ENEMY )
				local distance = Input.GetWorldCursorPos():__sub( Entity.GetAbsOrigin( enemy )):Length( )
				if distance > Menu.GetValue( BS.TerrorBladeTargetRadius ) then enemy = nil return end
			else
				local enemyes = Entity.GetHeroesInRadius( BS.Hero, Menu.GetValue( BS.TerrorBladeTargetRadius ), Enum.TeamType.TEAM_ENEMY)
				if not enemyes or #enemyes then return end
				enemy = enemyes[1]
				local distance = Entity.GetAbsOrigin( BS.Hero ):__sub( Entity.GetAbsOrigin( enemy )):Length( )
				if #enemyes > 1 then
					for i = 2, #enemyes do
						local d = Entity.GetAbsOrigin( BS.Hero ):__sub( Entity.GetAbsOrigin( enemyes[i] )):Length( )
						if d > distance then
							distance = d
							enemy = enemyes[i]
						end
					end
				end
			end
			BS.LockTarget( enemy )
			return
		end
		local distance = Entity.GetAbsOrigin( BS.Hero ):__sub( Entity.GetAbsOrigin( BS.Target )):Length( )
		local mana = NPC.GetMana( BS.Hero )
		if BS.E and Ability.IsCastable( BS.E, mana ) and distance < 550 then Ability.CastNoTarget( BS.E ) 
		elseif BS.Q and Ability.IsCastable( BS.Q, mana) and distance < 900 then Ability.CastNoTarget( BS.Q ) 
		elseif BS.W and Ability.IsCastable( BS.W, mana) then Ability.CastNoTarget( BS.W ) 
		elseif BS.Diffusal and Ability.IsReady( BS.Diffusal ) then Ability.CastTarget( BS.Diffusal, BS.Target ) 
		elseif BS.Manta and Ability.IsCastable( BS.Manta, mana) then Ability.CastNoTarget( BS.Manta ) 
		elseif BS.Bkb and Ability.IsReady( BS.Bkb ) then Ability.CastNoTarget( BS.Bkb ) 
		elseif BS.Orchid and Ability.IsCastable( BS.Orchid, mana) then Ability.CastTarget( BS.Orchid, BS.Target ) 
		elseif BS.Nullf and Ability.IsCastable( BS.Nullf, mana) then Ability.CastTarget( BS.Nullf, BS.Target ) 
		elseif BS.Mom and Ability.IsCastable( BS.Mom, mana) then Ability.CastNoTarget( BS.Mom ) end 
		BS.CT = BS.GT + 0.05
		Player.AttackTarget(Players.GetLocal(), BS.Hero, BS.Target, false)
	end
	function BS.TerrorBladeAutoSunder( ... )
		local mana 		= NPC.GetMana( BS.Hero )
		local health 	= Entity.GetHealth( BS.Hero )
		local maxHealth = Entity.GetMaxHealth( BS.Hero )
		local threshold = health / maxHealth * 100
		if not Ability.IsCastable( BS.R, mana ) then return end
		if threshold > Menu.GetValue( BS.TerrorBladeSunderThreshold ) then return end
		local radius = Ability.GetCastRange( BS.R )
		local enemyes = Entity.GetHeroesInRadius( BS.Hero, radius, Enum.TeamType.TEAM_ENEMY)
		if not enemyes or #enemyes == 0 then return end
		local enemy = enemyes[1]
		health = Entity.GetHealth( enemy ) / Entity.GetMaxHealth( enemy ) * 100
		if #enemyes > 1 then
			for i = 2, #enemyes do
				local iE = Entity.GetHealth( enemyes[i] ) / Entity.GetMaxHealth( enemyes[i] ) * 100
				if health > iE then
					health = iE
					enemy = enemyes[i]
				end
			end
		end
		Ability.CastTarget( BS.R, enemy )
	end

return BS
