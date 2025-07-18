<template>
    <div class="p-6 max-w-3xl mx-auto space-y-6">
        <h1 class="text-2xl font-bold">Vulkan4j Module Customizer</h1>

        <!-- 模块选择 -->
        <div>
            <label class="font-medium">Select Modules:</label>
            <div class="flex flex-wrap gap-2 mt-2">
                <label v-for="(name, key) in availableModules" :key="key" class="flex items-center gap-1">
                    <input type="checkbox" v-model="selectedModules" :value="key" />
                    {{ name }}
                </label>
            </div>
        </div>

        <div>
            <label class="font-medium">Version:</label>
            <select v-model="selectedVersion" class="ml-2 p-1 border rounded">
                <option v-for="version in versions" :key="version" :value="version">
                    {{ version }}
                </option>
            </select>
        </div>
        <!-- 构建工具选择 -->
        <div>
            <label class="font-medium">Build Tool:</label>
            <label class="ml-4"><input type="radio" value="gradle" v-model="buildTool" /> Gradle</label>
            <label class="ml-4"><input type="radio" value="maven" v-model="buildTool" /> Maven</label>
        </div>

        <!-- 生成按钮 -->
        <button class="mt-4 px-4 py-2 bg-blue-600 text-white rounded" @click="generateConfig">
            Generate Config
        </button>

        <!-- 结果展示 -->
        <pre
            class="bg-neutral-100 p-4 rounded-lg whitespace-pre-wrap text-left font-mono text-sm leading-relaxed overflow-x-auto border border-neutral-300">
<code class="text-left">{{ generatedCode }}</code>
        </pre>
    </div>
</template>

<script setup>
import { ref, computed,watch } from "vue";
import libraryJson from '../data/library.json' // 或 fetch 加载

const versions = ref(Object.keys(libraryJson.library.versions).sort((a, b) => b.localeCompare(a)));
const selectedVersion = ref(versions.value[0]);

const availableModules = computed(() => libraryJson.library.versions[selectedVersion.value] || []);
const selectedModules = ref([]);
const buildTool = ref("maven");
const useCustomRepo = ref(false);
const generatedCode = ref("");
const groupId = ref("club.doki7");


watch(selectedVersion, () => {
    selectedModules.value = selectedModules.value.filter((key) => availableModules.value[key] !== undefined);
});

function generateConfig() {
    const modules = selectedModules.value.sort().map((key) => {
    console.log(key, availableModules.value[key]);
        return availableModules.value[key];
    }).filter(value => value !== undefined);
    console.log(modules ,availableModules.value);
    if (buildTool.value === "gradle") {
        
    } else {
        generateMavenCode(modules);
    }
}

function generateMavenCode(modules) {
    const maven = [];
    if (useCustomRepo.value) {
        maven.push(`<repositories>`);
        maven.push(`  <repository>`);
        maven.push(`    <id>custom</id>`);
        maven.push(`    <url>https://your.repo/releases</url>`);
        maven.push(`  </repository>`);
        maven.push(`</repositories>\n`);
    }

    maven.push(`<dependencies>`);
    for (const mod of modules) {
        maven.push(`  <dependency>`);
        maven.push(`    <groupId>${groupId.value}</groupId>`);
        maven.push(`    <artifactId>${mod}</artifactId>`);
        maven.push(`    <version>${selectedVersion.value}</version>`);
        maven.push(`    <scope>compile</scope>`);
        maven.push(`  </dependency>`);
    }
    maven.push(`</dependencies>`);

    generatedCode.value = maven.join("\n");
}

</script>

<style scoped>
input[type="checkbox"],
input[type="radio"] {
    transform: scale(1.2);
}
</style>
