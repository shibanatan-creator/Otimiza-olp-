# Otimiza-olp-fiis
// build.gradle
plugins {
    id 'net.minecraftforge.gradle' version '5.1.+'
}

group = 'com.exemplo'
version = '1.0.0'

java {
    toolchain.languageVersion = JavaLanguageVersion.of(17)
}

minecraft {
    mappings channel: 'official', version: '1.20.1'
    
    runs {
        client {
            workingDirectory project.file('run')
            property 'forge.logging.console.level', 'debug'
            mods {
                optimizationmod {
                    source sourceSets.main
                }
            }
        }
    }
}

dependencies {
    minecraft 'net.minecraftforge:forge:1.20.1-47.2.0'
}

// ===== ARQUIVO: OptimizationMod.java =====
package com.exemplo.optimizationmod;

import net.minecraftforge.common.MinecraftForge;
import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.fml.event.lifecycle.FMLClientSetupEvent;
import net.minecraftforge.fml.event.lifecycle.FMLCommonSetupEvent;
import net.minecraftforge.fml.javafmlmod.FMLJavaModLoadingContext;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

@Mod("optimizationmod")
public class OptimizationMod {
    public static final String MOD_ID = "optimizationmod";
    private static final Logger LOGGER = LogManager.getLogger();

    public OptimizationMod() {
        FMLJavaModLoadingContext.get().getModEventBus().addListener(this::setup);
        FMLJavaModLoadingContext.get().getModEventBus().addListener(this::clientSetup);
        MinecraftForge.EVENT_BUS.register(this);
    }

    private void setup(final FMLCommonSetupEvent event) {
        LOGGER.info("Mod de Otimização inicializado!");
    }

    private void clientSetup(final FMLClientSetupEvent event) {
        LOGGER.info("Configurações de cliente aplicadas!");
    }
}

// ===== ARQUIVO: ChunkOptimizer.java =====
package com.exemplo.optimizationmod.optimization;

import net.minecraft.world.level.chunk.LevelChunk;
import net.minecraftforge.event.level.ChunkEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Mod.EventBusSubscriber(modid = "optimizationmod")
public class ChunkOptimizer {
    private static final Map<Long, Long> chunkAccessTimes = new ConcurrentHashMap<>();
    private static final long CACHE_TIME = 5000; // 5 segundos

    @SubscribeEvent
    public static void onChunkLoad(ChunkEvent.Load event) {
        if (event.getLevel().isClientSide()) {
            LevelChunk chunk = (LevelChunk) event.getChunk();
            long pos = chunk.getPos().toLong();
            chunkAccessTimes.put(pos, System.currentTimeMillis());
        }
    }

    @SubscribeEvent
    public static void onChunkUnload(ChunkEvent.Unload event) {
        if (event.getLevel().isClientSide()) {
            LevelChunk chunk = (LevelChunk) event.getChunk();
            chunkAccessTimes.remove(chunk.getPos().toLong());
        }
    }

    public static void cleanupOldChunks() {
        long currentTime = System.currentTimeMillis();
        chunkAccessTimes.entrySet().removeIf(entry -> 
            currentTime - entry.getValue() > CACHE_TIME
        );
    }
}

// ===== ARQUIVO: EntityOptimizer.java =====
package com.exemplo.optimizationmod.optimization;

import net.minecraft.world.entity.Entity;
import net.minecraft.world.entity.item.ItemEntity;
import net.minecraftforge.event.entity.EntityJoinLevelEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

import java.util.HashMap;
import java.util.Map;

@Mod.EventBusSubscriber(modid = "optimizationmod")
public class EntityOptimizer {
    private static final Map<Integer, Long> entityTickTimes = new HashMap<>();
    private static final int MAX_ENTITIES_PER_CHUNK = 50;

    @SubscribeEvent
    public static void onEntityJoin(EntityJoinLevelEvent event) {
        Entity entity = event.getEntity();
        
        // Otimiza itens no chão
        if (entity instanceof ItemEntity itemEntity) {
            // Aumenta o tempo de despawn se houver muitos itens
            if (countNearbyItems(itemEntity) > 20) {
                itemEntity.lifespan = 1200; // 1 minuto ao invés de 5
            }
        }
    }

    private static int countNearbyItems(ItemEntity item) {
        return item.level().getEntitiesOfClass(
            ItemEntity.class,
            item.getBoundingBox().inflate(8.0),
            e -> e != item
        ).size();
    }

    public static boolean shouldTickEntity(Entity entity) {
        int id = entity.getId();
        long currentTime = System.currentTimeMillis();
        Long lastTick = entityTickTimes.get(id);
        
        if (lastTick == null || currentTime - lastTick > 50) {
            entityTickTimes.put(id, currentTime);
            return true;
        }
        return false;
    }
}

// ===== ARQUIVO: RenderOptimizer.java =====
package com.exemplo.optimizationmod.optimization;

import net.minecraft.client.Minecraft;
import net.minecraftforge.client.event.RenderLevelStageEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = "optimizationmod")
public class RenderOptimizer {
    private static int frameSkipCounter = 0;
    private static final int FRAME_SKIP_THRESHOLD = 3;

    @SubscribeEvent
    public static void onRenderLevel(RenderLevelStageEvent event) {
        if (event.getStage() == RenderLevelStageEvent.Stage.AFTER_PARTICLES) {
            Minecraft mc = Minecraft.getInstance();
            
            // Reduz partículas se FPS baixo
            if (mc.getFps() < 30) {
                frameSkipCounter++;
                if (frameSkipCounter >= FRAME_SKIP_THRESHOLD) {
                    // Limita partículas
                    mc.particleEngine.clearParticles();
                    frameSkipCounter = 0;
                }
            } else {
                frameSkipCounter = 0;
            }
        }
    }

    public static int getOptimalRenderDistance() {
        Minecraft mc = Minecraft.getInstance();
        int fps = mc.getFps();
        
        if (fps >= 60) return 12;
        if (fps >= 45) return 10;
        if (fps >= 30) return 8;
        return 6;
    }
}

// ===== ARQUIVO: MemoryOptimizer.java =====
package com.exemplo.optimizationmod.optimization;

import net.minecraftforge.event.TickEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = "optimizationmod")
public class MemoryOptimizer {
    private static int tickCounter = 0;
    private static final int CLEANUP_INTERVAL = 1200; // 1 minuto

    @SubscribeEvent
    public static void onServerTick(TickEvent.ServerTickEvent event) {
        if (event.phase == TickEvent.Phase.END) {
            tickCounter++;
            
            if (tickCounter >= CLEANUP_INTERVAL) {
                performCleanup();
                tickCounter = 0;
            }
        }
    }

    private static void performCleanup() {
        // Limpa chunks antigos
        ChunkOptimizer.cleanupOldChunks();
        
        // Sugere coleta de lixo se memória estiver alta
        Runtime runtime = Runtime.getRuntime();
        long usedMemory = runtime.totalMemory() - runtime.freeMemory();
        long maxMemory = runtime.maxMemory();
        
        if (usedMemory > maxMemory * 0.85) {
            System.gc();
        }
    }

    public static String getMemoryInfo() {
        Runtime runtime = Runtime.getRuntime();
        long usedMB = (runtime.totalMemory() - runtime.freeMemory()) / 1024 / 1024;
        long maxMB = runtime.maxMemory() / 1024 / 1024;
        return String.format("Memória: %dMB / %dMB", usedMB, maxMB);
    }
}

// ===== ARQUIVO: mods.toml =====
modLoader="javafml"
loaderVersion="[47,)"
license="MIT"

[[mods]]
modId="optimizationmod"
version="1.0.0"
displayName="Optimization Mod"
description="Mod de otimização completo para Minecraft Java"
authors="Você"

[[dependencies.optimizationmod]]
modId="forge"
mandatory=true
versionRange="[47,)"
ordering="NONE"
side="BOTH"

[[dependencies.optimizationmod]]
modId="minecraft"
mandatory=true
versionRange="[1.20.1]"
ordering="NONE"
side="BOTH"
