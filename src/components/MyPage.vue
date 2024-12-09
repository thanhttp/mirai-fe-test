<template>
    <div>
        <div class="main">
            <div class="sidebar">
                <div class="storage">
                    <div class="d-flex mb-2">
                        <div>Storage</div>
                        <div class="ms-auto text-muted"><a href="#" class="text-reset">Change plan</a></div>
                    </div>
                    <div>
                        <input type="range" :value="usagePercentage" max="100" disabled class="w-100" id="range" />
                        <p class="text-start"><span class="text-primary">{{ usagePercentage }}%</span> used of 2GB</p>
                    </div>
                </div>
                <hr>
                <div class="search">
                    <div class="d-flex mb-2">
                        <div>Search</div>
                    </div>
                    <div>
                        <input type="text" v-model="searchQuery" placeholder="Search files..."
                            class="search-bar w-100 form-control" />
                    </div>
                </div>
                <hr>
                <div class="folders w-100">
                    <div class="d-flex mb-2">
                        <div>Folders</div>
                        <div class="ms-auto text-muted"><a href="#" class="text-reset">New Folder</a></div>
                    </div>
                    <div class="folder-tree">
                        <FolderTree :nodes="this.folders" @folder-clicked="handleFolderClick" :selectedFolder="selectedFolder" />
                    </div>
                </div>
                <hr>
                <div class="members">
                    <div class="d-flex mb-2">
                        <div>Members</div>
                        <div class="ms-auto text-muted"><a href="#" class="text-reset">Select all</a></div>
                    </div>
                    <ul class="ps-0">
                        <li v-for="member in members" :key="member" class="text-start ps-0">
                            <label>
                                <input type="checkbox" v-model="selectedMember" :value="member" />
                                {{ member }}
                            </label>
                        </li>
                    </ul>
                </div>
            </div>

            <div class="table">
                <b-table :items="filteredItems" :fields="fields" :sort-by.sync="sortBy" :sort-desc.sync="sortDesc"
                    responsive="sm">
                    <template #head(select)>
                        <b-form-checkbox v-model="selectAll" @change="toggleSelectAll"></b-form-checkbox>
                    </template>

                    <template #cell(select)="data">
                        <input type="checkbox" :value="data.item.id" :checked="isSelected(data.item)"
                            @change="toggleItemSelection(data.item)">
                    </template>
                    <template #cell(image)="data">
                        <img :src="data.item.image" alt="" class="w-100" style="height: 100px;" />
                    </template>
                </b-table>
            </div>
        </div>
    </div>
</template>

<script>
import FolderTree from './FolderTree.vue';
export default {
    components: {
        FolderTree,
    },
    data() {
        return {
            sortBy: 'name',
            sortDesc: false,
            selectedItems: [],
            selectAll: false,
            fields: [
                { key: 'select', label: '', sortable: false },
                { key: 'image', label: 'Select All', sortable: false },
                { key: 'name', sortable: true },
                { key: 'dimmension', sortable: true },
                { key: 'size', sortable: true },
            ],
            items: [],

            storageLimit: 2 * 1024,
            folders: [],
            members: ["All", "Admin", "Apple"],
            selectedFolder: 1,
            selectedMember: ["All"],
            searchQuery: "",
        }
    },
    computed: {
        isAllSelected() {
            return this.selectedItems.length === this.items.length;
        },
        filteredItems() {
            const currentFolder = this.findFolderById(this.folders, this.selectedFolder);

            if (!currentFolder) return [];

            let items = this.getAllFiles(currentFolder);

            if (!this.selectedMember.includes("All")) {
                items = items.filter((item) =>
                    item.photo_by && this.selectedMember.includes(item.photo_by)
                );
            }

            return this.searchQuery
                ? items.filter(item => item.name.toLowerCase().includes(this.searchQuery.toLowerCase()))
                : items;
        },
        usagePercentage() {
            const calculateSize = (items) => {
                return items.reduce((sum, item) => {
                    let itemSize = Number(item.size || 0);
                    let childrenSize = Number(item.children ? calculateSize(item.children) : 0);
                    return sum + itemSize + childrenSize;
                }, 0);
            };
            let usedSpace = calculateSize(this.folders);

            return ((usedSpace / 10000 / this.storageLimit) * 100).toFixed(1);
        },
    },
    watch: {
        selectedItems(newVal) {
            this.selectAll = newVal.length === this.filteredItems.length && newVal.length > 0;
        }
    },
    methods: {
        findFolderById(folders, id) {
            for (let folder of folders) {
                if (folder.id === id) {
                    return folder;  
                }
                if (folder.children && folder.children.length > 0) {
                    const foundInChildren = this.findFolderById(folder.children, id);
                    if (foundInChildren) {
                        return foundInChildren;  
                    }
                }
            }
            return null;  
        },
        handleFolderClick(folderId) {
            console.log("Folder clicked:", folderId);
            this.selectedFolder = folderId;
        },
        toggleSelectAll() {
            if (this.selectAll) {
                this.selectedItems = [...this.filteredItems];
            } else {
                this.selectedItems = [];
            }
        },

        isSelected(item) {
            return this.selectedItems.some(selected => selected.id === item.id);
        },

        toggleItemSelection(item) {
            const index = this.selectedItems.findIndex(selected => selected.id === item.id);

            if (index === -1) {
                this.selectedItems.push(item);
            } else {
                this.selectedItems.splice(index, 1);
            }

            this.selectAll = this.selectedItems.length === this.filteredItems.length;
        },
        selectFolder(folderId) {
            this.selectedFolder = folderId;
        },
        async fetchData() {

            this.folders = [
                {
                    "id": 1,
                    "name": "Uploads",
                    "children": [
                        {
                            "id": 10,
                            "name": "Images",
                            "children": [
                                {
                                    "id": 101,
                                    "name": "Seasandiego.jpg",
                                    "url": "https://assets.site-static.com/userFiles/2597/image/2023/CARDIFF_BY_THE_SEA/5-23-2023_1__reasons-why-cardiff-by-the-sea-san-diego-great-place-to-live/9_Reasons_Why_Cardiff-by-the-Sea_San_Diego_.jpg",
                                    "type": "image/jpg",
                                    "dimmension": "2000x2000",
                                    "size": "763300",
                                    "photo_by": "Admin"
                                },
                                {
                                    "id": 102,
                                    "name": "iMactwin.png",
                                    "url": "https://i.insider.com/60e4cb0d22d19400191c957c?width=1000&format=jpeg&auto=webp",
                                    "type": "image/png",
                                    "dimmension": "2000x2000",
                                    "size": "640200",
                                    "photo_by": "Apple"
                                },
                                {
                                    "id": 103,
                                    "name": "hustlecup.jpg",
                                    "url": "https://images.deliveryhero.io/image/fd-ph/LH/kmph-hero.jpg",
                                    "type": "image/jpg",
                                    "dimmension": "2000x2000",
                                    "size": "100400",
                                    "photo_by": "Admin"
                                },
                                {
                                    "id": 104,
                                    "name": "MS-Surface.jpg",
                                    "url": "https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RE4OXzi?ver=3a58",
                                    "type": "image/jpg",
                                    "dimmension": "2000x2000",
                                    "size": "113200",
                                    "photo_by": "Admin"
                                },
                                {
                                    "id": 105,
                                    "name": "Famoustower.jpg",
                                    "url": "https://www.worldfamousthings.com/wp-content/uploads/2018/01/Leaning-Tower-of-Pisa-Italy.jpg",
                                    "type": "image/jpg",
                                    "dimmension": "2000x2000",
                                    "size": "763300",
                                    "photo_by": "Admin"
                                }
                            ]
                        },
                        {
                            "id": 20,
                            "name": "Document",
                            "children": []
                        },
                        {
                            "id": 30,
                            "name": "Videos",
                            "children": [
                                {
                                    "id": 301,
                                    "name": "GODZILLA MINUS ONE Official Trailer",
                                    "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/No-Image-Placeholder.svg/330px-No-Image-Placeholder.svg.png?20200912122019",
                                    "type": "video/mp4",
                                    "dimmension": "2000x2000",
                                    "size": "763300",
                                    "photo_by": "Admin"
                                },
                                {
                                    "id": 302,
                                    "name": "Marvel Studios’ Loki Season 2 | October 6 on Disney+",
                                    "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/No-Image-Placeholder.svg/330px-No-Image-Placeholder.svg.png?20200912122019",
                                    "type": "video/mp4",
                                    "dimmension": "2000x2000",
                                    "size": "763300",
                                    "photo_by": "Admin"
                                },
                                {
                                    "id": 303,
                                    "name": "THE BOY AND THE HERON | Official Teaser Trailer",
                                    "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/No-Image-Placeholder.svg/330px-No-Image-Placeholder.svg.png?20200912122019",
                                    "type": "video/mp4",
                                    "dimmension": "2000x2000",
                                    "size": "763300",
                                    "photo_by": "Admin"
                                }
                            ]
                        }
                    ]
                },
                {
                    "id": 2,
                    "name": "Sources",
                    "children": []
                },
                {
                    "id": 3,
                    "name": "Shared",
                    "children": []
                }
            ];

            this.selectedFolder = this.folders[0]?.id;
        },
        getAllFiles(folder) {
            let files = [];
            if (folder.children) {
                folder.children.forEach(child => {
                    if (child.children) {
                        files = [...files, ...this.getAllFiles(child)];
                    } else {
                        files.push({
                            id: child.id,
                            isActive: true,
                            size: child.size,
                            name: child.name,
                            dimmension: child.dimmension,
                            image: child.url,
                            photo_by: child.photo_by,
                        });
                    }
                });
            }

            return files;
        },
    },
    created() {
        this.fetchData();
    },
}
</script>


<style>
.main {
    display: flex;
    height: 100vh;
}

.sidebar {
    width: 25%;
    padding: 1rem;
    background: #f5f5f5;
}

.table {

    flex: 1;
    padding: 1rem;
    background: #fff;
}

.storage p {
    margin: 0;
}


ul {
    list-style: none;
    padding: 0;
}

ul li {
    cursor: pointer;
    /* margin: 0.5rem 0; */
}

ul li.active {
    font-weight: bold;
    color: blue;
}

.file-list {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.file-item {
    width: calc(33.33% - 1rem);
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.folders .folder-tree div:nth-of-type(1)>ul {
    padding-left: 0;
}

#range {
    -webkit-appearance: none;
    appearance: none;
    width: 100%;
    cursor: pointer;
    outline: none;
    overflow: hidden;
    border-radius: 16px;
}

#range::-webkit-slider-runnable-track {
    height: 15px;
    background: #ccc;
    border-radius: 16px;
}

#range::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    height: 15px;
    width: 15px;
    background-color: #0a58ca;
    border-radius: 50%;
    border: 2px solid #0a58ca;
    box-shadow: -407px 0 0 400px #0a58ca;
}

#range::-moz-range-thumb {
    height: 15px;
    width: 15px;
    background-color: #0a58ca;
    border-radius: 50%;
    border: 1px solid #0a58ca;
    box-shadow: -407px 0 0 400px #0a58ca;
}

.sr-only {
    display: none;
}
</style>