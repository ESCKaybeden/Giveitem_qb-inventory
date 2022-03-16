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
        local closestPlayer, closestDistance = QBCore.Functions.GetPlayersFromCoords(GetEntityCoords(PlayerPedId()), 3)
        local Players = {}
        for sayi,esc in pairs(closestPlayer) do
            if GetPlayerServerId(esc) ~= GetPlayerServerId(PlayerId()) then
                QBCore.Functions.TriggerCallback('GetPlayers', function(props)
                    table.insert(Players, {props = props, id = GetPlayerServerId(esc)})
                    cb({Players = Players,Item = data.item})
                end, GetPlayerServerId(esc))
            end
        end
    end)

    RegisterNUICallback("GiveItem1", function(data, cb)
        local closestPlayer, closestDistance = QBCore.Functions.GetPlayersFromCoords(GetEntityCoords(PlayerPedId()), 3)
        for sayi,esc in pairs(closestPlayer) do
            if GetPlayerServerId(esc) == data.player then
                SetCurrentPedWeapon(playerPed,'WEAPON_UNARMED',true)
                TriggerServerEvent("inventory:server:GiveItem0", data.player, data.item, data.amount, data.item.slot)
            else
                QBCore.Functions.Notify("Failed", "Yakınında kimse yok!")
            end
        end
    end)
```

#### Server

**_server/main.lua_** dosyasını açıp en alt satıra yapıştırın
```lua
    QBCore.Functions.CreateCallback('GetPlayers', function(source,cb,id)
      local Player = QBCore.Functions.GetPlayer(id).PlayerData.charinfo
      cb(Player.firstname .. " " .. Player.lastname)
    end)

    RegisterServerEvent("inventory:server:GiveItem0", function(target, name, amount, slot)
        local src = source
        local Player = QBCore.Functions.GetPlayer(src)
        local OtherPlayer = QBCore.Functions.GetPlayer(tonumber(target))
        local dist = #(GetEntityCoords(GetPlayerPed(src))-GetEntityCoords(GetPlayerPed(target)))
      if Player == OtherPlayer then return TriggerClientEvent('QBCore:Notify', src, "Kendine bir eşya veremez misin?") end
      if dist > 2 then return TriggerClientEvent('QBCore:Notify', src, "Eşya vermek için çok uzaktasın!") end
      local item = Player.Functions.GetItemBySlot(slot)
      if item.name ~= name.name then TriggerClientEvent('QBCore:Notify', src, "Yanlış öğe bulundu tekrar deneyin!"); return end
      if amount <= item.amount then
        if amount == 0 then	amount = item.amount end
        if Player.Functions.RemoveItem(item.name, amount, item.slot) then
          if OtherPlayer.Functions.AddItem(item.name, amount, false, item.info) then
            TriggerClientEvent('inventory:client:ItemBox',target, QBCore.Shared.Items[item.name], "add")
            TriggerClientEvent('QBCore:Notify', target, Player.PlayerData.charinfo.firstname.." "..Player.PlayerData.charinfo.lastname.. " tarafından "..amount..' adet '..item.label.." aldınız")
            TriggerClientEvent("inventory:client:UpdatePlayerInventory", target, true)
            TriggerClientEvent('inventory:client:ItemBox',src, QBCore.Shared.Items[item.name], "remove")
            TriggerClientEvent('QBCore:Notify', src, "" .. OtherPlayer.PlayerData.charinfo.firstname.." "..OtherPlayer.PlayerData.charinfo.lastname.. "'e " .. amount .. " adet " .. item.label .." verdiniz!")
            TriggerClientEvent("inventory:client:UpdatePlayerInventory", src, true)
            TriggerClientEvent('qb-inventory:client:giveAnim', src)
            TriggerClientEvent('qb-inventory:client:giveAnim', target)
          else
            Player.Functions.AddItem(item.name, amount, item.slot, item.info)
            TriggerClientEvent('QBCore:Notify', src,  "Diğer oyuncuların envanteri dolu!", "error")
            TriggerClientEvent('QBCore:Notify', target,  "Envanterin dolu!", "error")
            TriggerClientEvent("inventory:client:UpdatePlayerInventory", src, false)
            TriggerClientEvent("inventory:client:UpdatePlayerInventory", target, false)
          end
        else
          TriggerClientEvent('QBCore:Notify', src,  "Yeterince öğeniz yok", "error")
        end
      else
        TriggerClientEvent('QBCore:Notify', src, "Aktarmak için yeterli öğeniz yok")
      end
    end)
```


#### JavaScript

**_html/js/app.js_** dosyasını açıp crtl + f tuşlarına basıp sağ üst tarafta açılan arama kısmına "#item-give" yazıp çıkan kısımda ki kodu değiştirin
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

html içerisindeki **_ui.html_** dosyasını açıp id="qbcore-inventory" kısmının bir alt satırında boş yer oluşturup yapıştırın
```html
    <div class="GiveItembox">
        <div class="GiveItemPlayers"></div>
    </div>
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
        /* backdrop-filter: blur(.2em); */
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
    .GiveItemPlayersButton:hover {
        background-color: rgba(34, 29, 46, 0.8);
    }
```




## English (very soon..)


