<template>
    <div class="w-100">
        <ul>
            <li v-for="node in nodes" :key="node.id + '-folder-id'">
                <div v-if="node.children" @click="toggleExpand(node)">
                    <span>{{ expanded[node.id] ? "[-]" : "[+]" }}</span>
                    {{ node.name }}
                </div>
                <FolderTree v-if="node.children && node.children.length > 0 && expanded[node.id]"
                    :nodes="node.children" @folder-clicked="handleFolderClick" />
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

li div {
    cursor: pointer;
}
</style>
