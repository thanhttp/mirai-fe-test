<template>
    <div class="w-100">
        <ul>
            <li v-for="node in nodes" :key="node.id + '-folder-id'">
                <div v-if="node.children" @click="toggleExpand(node)" :class="{ active: node.id === selectedFolder }" class="rounded">
                    <span>{{ expanded[node.id] ? "[-]" : "[+]" }}</span>
                    {{ node.name }}
                </div>
                <FolderTree v-if="node.children && node.children.length > 0 && expanded[node.id]" :nodes="node.children"
                    @folder-clicked="handleFolderClick" :selectedFolder="selectedFolder"/>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    name: "FolderTree",
    props: {
        nodes: {
            type: Array,
            required: true,
        },
        selectedFolder: {
            type: Number,
            default: null,
        },
    },
    data() {
        return {
            expanded: {},
        };
    },
    methods: {
        toggleExpand(node) {
            this.$emit("folder-clicked", node.id);

            this.$set(this.expanded, node.id, !this.expanded[node.id]);
        },
        handleFolderClick(folderId) {
            this.$emit("folder-clicked", folderId);
        }
    },
};
</script>

<style scoped>
ul {
    list-style: none;
    padding-left: 20px;
}

li {
    text-align: left;
}

li > div:nth-child(1){
    cursor: pointer;
    padding-top: 10px;
    padding-bottom: 10px;
}

li div span{
    padding-left: 10px;
}

li div.active {
  background-color: #a6c8ee;
  color: var(--bs-primary);
  font-weight: bold;
}

</style>
