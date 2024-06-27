import { world, system } from "@minecraft/server";
import { ActionFormData, ModalFormData } from "@minecraft/server-ui";

/**
 # INVSEE
 * @writeBy @a2good
 * {@link https://A2good611.github.io}
 * command:
 * +invsee <playerName>
 * +invsee OpenUI
 */

async function forceOpen(player, form) {
    while (true) {
        const f = await form.show(player);
        if (f.cancelationReason !== "UserBusy") return f;
    }
}

world.beforeEvents.chatSend.subscribe(ev => {
    const { sender: ply, message: m } = ev;

    function BilekUI(player) {
        const plyList = world
                .getAllPlayers()
                .map(v => v.name) /** @returns {Player.name[]} */, //jaga" kaloo semisal Player offline
            ui = new ModalFormData()
                .title("§l[ INVSEE ]")
                .dropdown("TARGET:", plyList);
        forceOpen(player, form).then(({ canceled, formValues: value }) => {
            if (canceled) return;
            if (!mc.world.getAllPlayers().find(v => v.name === plyList[value]))
                return player.sendMessage("Player Offline");

            invseeUI(player, plyList[value]);
        });
    }

    function invseeUI(player, targetName) {
        const target = world.getPlayers({ name: targetName })[0],
            inv = target.getComponent("inventory"),
            container = inv.container,
            { size: invSize, emptySlotsCount: emptySlot } = container,
            sukiMeg = new ActionFormData()
                .title(`§l[ INVSEE ${targetName} ]`)
                .body(`slot: ${invSize - emptySlot}/${invSize}`);
        for (let i = 0; i < invSize; i = i + 1) {
            let item = container.getItem(i);
            if (item === undefined)
                item = {
                    typeId: "n:none",
                    amount: 0,
                    nameTag: ""
                };

            sukiMeg.button(
                `§a2[§a${i}§2] §f${
                    item.nameTag ? item.nameTag : `§b${item.typeId}`
                }\n§eamount: §g${item.amount}`
            );
        }
        forceOpen(player, sukiMeg);
    }
    if (m.startsWith("+invsee")) {
        const args = m.slice(7).split(" "),
            targetName = args[0];

        if (!targetName) BilekUI(ply);
        else invseeUI(ply, targetName);
    }
});
