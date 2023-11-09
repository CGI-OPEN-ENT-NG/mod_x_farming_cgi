# mod_x_farming_cgi
Projet fork depuis https://bitbucket.org/minetest_gamers/x_farming/src/master/ pour appliquer correctif


# Correctif
Afin d'éviter un crash du serveur minetest pour un nil value, noua ajoutons une vérification dans le code du fichier api.lua
L 2458 :

```
function x_farming.get_spawn_pos_abr(dtime, intrvl, radius, chance, reduction)
    dtime = math.min(dtime, 0.1)
    local players = minetest.get_connected_players()
    intrvl = 1 / intrvl

    if math.random() < dtime * (intrvl * #players) then
        -- choose random player
        local player = players[math.random(#players)]
        local vel = player:get_velocity()

        -- Ajoute une vérification pour s'assurer que vel n'est pas nul
        if vel then
            local spd = vector.length(vel)
            chance = (1 - chance) * 1 / (spd * 0.75 + 1)

            local yaw
            if spd > 1 then
                -- spawn in the front arc
                yaw = minetest.dir_to_yaw(vel) + math.random() * 0.35 - 0.75
            else
                -- random yaw
                yaw = math.random() * math.pi * 2 - math.pi
            end

            local pos = player:get_pos()
            local dir = vector.multiply(minetest.yaw_to_dir(yaw), radius)
            local pos2 = vector.add(pos, dir)

            pos2.y = pos2.y - 5

            local height, liquidflag = x_farming.get_terrain_height(pos2, 32)
            if height then
                local objs = minetest.find_node_near(pos, radius * 1.1, { 'group:bee' }) or {}

                -- count mobs in abrange
                for _, obj in ipairs(objs) do
                    chance = chance + (1 - chance) * reduction
                end

                if chance < math.random() then
                    pos2.y = height
                    objs = minetest.get_objects_inside_radius(pos2, radius * 0.95)

                    -- do not spawn if another player around
                    for _, obj in ipairs(objs) do
                        if obj:is_player() then
                            return
                        end
                    end

                    return pos2, liquidflag
                end
            end
        end
    end
end

```
