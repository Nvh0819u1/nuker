package example.addon.modules;

import minegame159.meteorclient.events.world.TickEvent;
import minegame159.meteorclient.systems.modules.Module;
import minegame159.meteorclient.utils.player.Rotations;
import minegame159.meteorclient.utils.world.BlockUtils;
import net.minecraft.network.packet.c2s.play.PlayerActionC2SPacket;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Direction;
import net.minecraft.util.Hand;
import meteordevelopment.orbit.EventHandler;
import example.addon.Addon;
import java.util.ArrayList;
import java.util.List;

public class UltraNuker extends Module {
    public UltraNuker() {
        super(Addon.CATEGORY, "ultra-nuker", "Phá tối đa 3 khối mỗi tick bằng Packet Bypass.");
    }

    @EventHandler
    private void onTick(TickEvent.Pre event) {
        List<BlockPos> targets = new ArrayList<>();
        BlockUtils.getSphere(mc.player.getBlockPos(), 5, 5).forEach(p -> {
            if (!mc.world.getBlockState(p).isAir() && canInstaBreak(p)) targets.add(p);
        });

        int broken = 0;
        for (BlockPos pos : targets) {
            if (broken >= 3) break;
            breakBlock(pos);
            broken++;
        }
    }

    private void breakBlock(BlockPos pos) {
        Rotations.rotate(Rotations.getYaw(pos), Rotations.getPitch(pos), () -> {
            mc.getNetworkHandler().sendPacket(new PlayerActionC2SPacket(PlayerActionC2SPacket.Action.START_DESTROY_BLOCK, pos, Direction.UP));
            mc.getNetworkHandler().sendPacket(new PlayerActionC2SPacket(PlayerActionC2SPacket.Action.STOP_DESTROY_BLOCK, pos, Direction.UP));
            mc.player.swingHand(Hand.MAIN_HAND);
        });
    }

    private boolean canInstaBreak(BlockPos pos) {
        float hardness = mc.world.getBlockState(pos).getHardness(mc.world, pos);
        return hardness >= 0 && hardness <= 0.4;
    }
}
