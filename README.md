# MixinBooter with Ancient Mixin Compatibility

### What is this fork?

MixinBooter is designed to load Mixins in a modern way. Unfortunately this causes a number of old Mixin implementing mods to crash the game.

The source of the problem is caused by classes org.spongepowered.asm.launch.MixinBootstrap and org.spongepowered.asm.mixin.Mixins. Both of these are commonly accessed in old code and also in MixinBooter. Unfortunately due to how forge loads CoreMod Plugins, there is a number of mods that access these classes before MixinBooter which causes MixinBooter to fail to load properly and results in a crash.

The solution I have created is rather scuffed. I have renamed the "MixinBootstrap" and "Mixins" classes to "TrueMixinBootstrap" and "TrueMixins" in the UniMix implementation used by this mod. In addition I have replaced the old "MixinBootstrap" and "Mixins" classes with custom ones that implement only the commonly called methods: MixinBootstrap.init(), Mixins.addConfiguration(), and Mixins.addConfigurations().

Any mixin configurations added before the MixinBooter CorePlugin starts will be saved to memory and then loaded when MixinBooter calls Mixins.activate(TrueMixins.bridge) which successfully add old mod mixin configs without issue.

Only MixinBooter calls the TrueMixinBootstrap.init() method, ensuring that no other mixin mod initializes the Bootstrap at the wrong time.

### How tested is this?

I've tested this on a modpack containing my own Mixin mod (using the old way) as well as with UniversalTweaks. Both mixins were successfully loaded, with my Mixins.addConfiguration() calls occuring before the MixinBooter initializations. Sponge compatibility still needs to be tested however. 



### Allows any mixins that work on mods to work effortlessly on 1.8 - 1.12.2

- Current Mixin Version: [UniMix 0.12.2 forked by CleanroomMC, derived from 0.8.5 branch by LegacyModdingMC](https://github.com/CleanroomMC/UniMix)

- Current MixinExtra Version: [0.2.1-beta.2](https://github.com/LlamaLad7/MixinExtras)

### For Developers:

- Add CleanroomMC's repository and depend on MixinBooter's maven entry:

```groovy
repositories {
    maven {
        url 'https://maven.cleanroommc.com'
    }
}

dependencies {

    // Common:
    annotationProcessor 'org.ow2.asm:asm-debug-all:5.2'
    annotationProcessor 'com.google.guava:guava:32.1.2-jre'
    annotationProcessor 'com.google.code.gson:gson:2.8.9'

    // ForgeGradle:
    implementation ('zone.rong:mixinbooter:8.9') {
        transitive = false
    }
    annotationProcessor ('zone.rong:mixinbooter:8.9') {
        transitive = false
    }
    
    // RetroFuturaGradle:
    String mixinBooter = modUtils.enableMixins('zone.rong:mixinbooter:8.9')
    // modUtils.enableMixins('zone.rong:mixinbooter:8.9', 'mod_id.mixins.refmap.json') << add refmap name as 2nd arg (optional)
    api (mixinBooter) {
        transitive = false
    }
    annotationProcessor (mixinBooter) {
        transitive = false
    }
}
```

### Pseudo-Changelog:

- As of 4.2, MixinBooter's API has changed and ***all mods*** that uses mixins are encouraged to depend on MixinBooter, even those that mixin into vanilla/forge/library classes. To avoid mixin version mismatches with mods crashing trying to implement modded mixins (looking at you VanillaFix). Thanks to [@embeddedt](https://github.com/embeddedt) recommending and helping me introduce this change!

- As of 5.0, [MixinExtras by @LlamaLad7](https://github.com/LlamaLad7/MixinExtras) is shaded. Available for developers to use.

- As of 8.0, MixinBooter will now work from 1.8 - 1.12.2. One single build works with all these versions! (TODO: LiteLoader support?)

- As of 8.4, MixinBooter actively attempts to be compatible with [SpongeForge](https://github.com/SpongePowered/SpongeForge)

### Tidbits:

- Consult `IEarlyMixinLoader` for mixins that affects vanilla, forge, or any classes that is passed to the classloader extremely early (e.g. Guava).
- Consult `ILateMixinLoader` for mixins that affects mods.
- `@MixinLoader` annotation is, as of 4.2, deprecated. The functionality is akin to `ILateMixinLoader`.
