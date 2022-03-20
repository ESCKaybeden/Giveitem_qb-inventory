<div align="center">
  <h1>ESCKaybeden</h1>
</div>

- **_ESCKaybeden#0488_**
- [**_Youtube_**](https://www.youtube.com/channel/UCwmyBjDNow69-4A2jCRe4Sg)
- [**Discord**](https://discord.gg/2drcthqyAF)

## Türkçe


#### Client

**_client/main.lua_** dosyasını açıp en alt satıra yapıştırın
```lua
RegisterNUICallback("GiveItem0", function(data, cb)
    local closestPlayer, closestDistance = ESX.Game.GetPlayersInArea(GetEntityCoords(PlayerPedId()), 3)
    local Players = {}
    for sayi,esc in pairs(closestPlayer) do
        if GetPlayerServerId(esc) ~= GetPlayerServerId(PlayerId()) then
            ESX.TriggerServerCallback('GetPlayers', function(props)
                table.insert(Players, {props = props, id = GetPlayerServerId(esc)})
                cb({Players = Players,Item = data.item})
            end, GetPlayerServerId(esc))
        end
    end
end)

RegisterNUICallback("GiveItem1", function(data, cb)
    local closestPlayer, closestDistance = ESX.Game.GetPlayersInArea(GetEntityCoords(PlayerPedId()), 3)
    for sayi,esc in pairs(closestPlayer) do
        if GetPlayerServerId(esc) ~= data.player then
            SetCurrentPedWeapon(playerPed,'WEAPON_UNARMED',true)
            TriggerServerEvent("inventory:server:GiveItem0", data.player, data.item, data.amount)
        end
    end
end)
```

#### Server

**_server/main.lua_** dosyasını açıp en alt satıra yapıştırın
```lua
ESX.RegisterServerCallback('GetPlayers', function(source,cb,id)
	exports.ghmattimysql:execute( "SELECT * FROM `users` WHERE `identifier` = '"..ESX.GetPlayerFromId(id).identifier.."'", function(Player)
		if Player[1] ~= nil then
			cb(Player[1].firstname .. " " .. Player[1].lastname)
		end
	end)
end)

RegisterServerEvent("inventory:server:GiveItem0", function(target, name, amount)
    local src = source
    local Player = ESX.GetPlayerFromId(src)
    local OtherPlayer = ESX.GetPlayerFromId(tonumber(target))
    local dist = #(GetEntityCoords(GetPlayerPed(src))-GetEntityCoords(GetPlayerPed(target)))
	if Player == OtherPlayer then return TriggerClientEvent('QBCore:Notify', src, "You can't give yourself an item?") end
	if dist > 2 then return TriggerClientEvent('QBCore:Notify', src, "You are too far away to give items!") end
	if amount <= name.count then
		if amount == 0 then
			amount = name.amount
		end
		if OtherPlayer.addInventoryItem(name.name, amount, false, name.info) then
			TriggerClientEvent('inventory:client:ItemBox',target, ESX.Items[name.name], "add")
			TriggerClientEvent("inventory:client:UpdatePlayerInventory", target, true)
			Player.removeInventoryItem(name.name, amount, name.slot)
			TriggerClientEvent('inventory:client:ItemBox',src, ESX.Items[name.name], "remove")
			TriggerClientEvent("inventory:client:UpdatePlayerInventory", src, true)
			TriggerClientEvent('qb-inventory:client:giveAnim', src)
			TriggerClientEvent('qb-inventory:client:giveAnim', target)
		else
			TriggerClientEvent("inventory:client:UpdatePlayerInventory", src, false)
			TriggerClientEvent("inventory:client:UpdatePlayerInventory", target, false)
		end
	end
end)
```


#### JavaScript

**_html/js/app.js_** dosyasını açıp en alt satıra yapıştırın
```js
$("#item-give").droppable({hoverClass: "button-hover",
    drop: function(event, ui) {
        setTimeout(function() { IsDragging = false; }, 300);
        fromData = ui.draggable.data("item");
        fromInventory = ui.draggable.parent().attr("data-inventory");
        amount = $("#item-amount").val();
        if (amount == 0) {
            return;
        } else {
            amount = fromData.amount;
            $.post(`https://${GetParentResourceName()}/GiveItem0`, JSON.stringify({ inventory: fromInventory, item: fromData, amount: parseInt(amount)
            }), function(state) { $('.GiveItemPlayers').html("")
                state.Players.map(dt => $(".GiveItemPlayers").append('<div class="GiveItemPlayersButton" data-player="' + dt.id + '">' + dt.props + ' (' + dt.id + ')</div>'))
                $(".GiveItembox").css({'display': 'block'});
                $(".GiveItemPlayersButton").click(function () { $(".GiveItembox").css({'display': 'none'});
                    player = $(this).data("player");
                    new Promise(function () {
                        const https = new XMLHttpRequest();
                        https.open("POST", `https://${GetParentResourceName()}/GiveItem1`);
                        https.send(JSON.stringify({
                            player: player,
                            item: state.Item,
                            amount: parseInt($("#item-amount").val())
                        }));
                    });
                });
            });
        }
    },
});
```


#### html

html içerisindeki **_ui.html_** dosyasını açıp **_id="qbcore-inventory"_** kısmının bir alt satırında boş yer oluşturup yapıştırın
```html
<div class="GiveItembox">
    <div class="GiveItemPlayers"></div>
</div>
```

**_id="item-use"_** kısmının bir alt satırına yapıştırın
```html
<div class="inv-option-item" id="item-give"><p>Ver</p></div>
```


#### css

**_html/css/main.css_** dosyasını açıp en alt satıra yapıştırın

```css
.GiveItembox {
    display: none;
    position: absolute;
    height: 13em;
    width: 16em;
    margin: auto;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    background: #1e1e1eaa;
    border-radius: .2em;
    z-index: 560;
    overflow-y: scroll;
}

.GiveItemPlayersButton {
    width: 100%;
    text-decoration: none;
    color: rgba(255, 255, 255, 0.719);
    background-color: #3030305a;
    text-shadow: none; 
    font-size: 14px !important;
    outline: none;
    text-transform: none;
    text-align: center;
    line-height: 30px;
    border: none;
    border-bottom: 1px solid #46464676;
    transition: 300ms all;
}
.GiveItemPlayersButton:hover { background-color: rgba(34, 29, 46, 0.8); }
```




## English (very soon..)





<div align="center">
  <h1>🎉 Result 🎉</h1>
</div>
<img align="center" alt="Game Image - 1" src="https://cdn.discordapp.com/attachments/833938679125770240/953446729154445322/unknown.png"/>
<img align="center" alt="Game Image - 2" src="https://cdn.discordapp.com/attachments/833938679125770240/953450782966054922/unknown.png"/>


