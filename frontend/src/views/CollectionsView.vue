<script setup>
/**
 * @file CollectionsView.vue
 * @description A comprehensive management interface for image collections, supporting static and dynamic (smart) groupings aligned with the Latent Design System.
 */
import { ref, onMounted, computed, watch } from 'vue';
import api, { authenticatedUrl } from '@/services/api';
import { useBrowserStore } from '@/stores/browser';
import LButton from '@/components/ds/LButton.vue';
import LSwitch from '@/components/ds/LSwitch.vue';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import Dropdown from 'primevue/dropdown';
import MultiSelect from 'primevue/multiselect';
import { useToast } from 'primevue/usetoast';
import { useRouter } from 'vue-router';
import CustomContextMenu from '@/components/CustomContextMenu.vue';
import { Plus, Folder, Bolt, Check, X, FolderOpen } from 'lucide-vue-next';

const store = useBrowserStore();
const router = useRouter();
const toast = useToast();

const collections = ref([]);
const displayCreateDialog = ref(false);
const isEditing = ref(false);

const cm = ref();
const menuModel = ref([]);
const contextMenuSelection = ref(null);

const newCollectionName = ref('');
const originalCollectionName = ref('');
const isSmartCollection = ref(false);
const prompt = ref('');
const selectedModels = ref([]);
const selectedLoras = ref([]);
const selectedSamplers = ref([]);
const selectedRating = ref(null);

const ratingOptions = [
  { label: 'None (Unrated)', value: 0 },
  { label: 'Any Star (> 0)', value: 'Any Star Count' },
  { label: '1 Star', value: 1 },
  { label: '2 Stars', value: 2 },
  { label: '3 Stars', value: 3 },
  { label: '4 Stars', value: 4 },
  { label: '5 Stars', value: 5 }
];

watch(() => store.navRefreshKey, () => {
  fetchCollections();
});

watch(() => store.collectionToEdit, (newName) => {
  if (newName) {
    editCollection(newName);
    store.collectionToEdit = null;
  }
});

const refreshFilters = async () => {
  await store.loadFilters();
};

const resetForm = () => {
  newCollectionName.value = '';
  originalCollectionName.value = '';
  isSmartCollection.value = false;
  prompt.value = '';
  selectedModels.value = [];
  selectedLoras.value = [];
  selectedSamplers.value = [];
  selectedRating.value = null;
  isEditing.value = false;
};

const openCreateDialog = () => {
  resetForm();
  displayCreateDialog.value = true;
};

const editCollection = async (name) => {
  try {
    const response = await api.get(`/collections/${encodeURIComponent(name)}`);
    const data = response.data;

    newCollectionName.value = data.name;
    originalCollectionName.value = data.name;
    isSmartCollection.value = data.isSmart;

    if (data.filters) {
      selectedModels.value = data.filters.models || [];
      selectedLoras.value = data.filters.loras || [];
      selectedSamplers.value = data.filters.samplers || [];

      if (data.filters.rating === '0') selectedRating.value = 0;
      else if (data.filters.rating === 'Any Star Count') selectedRating.value = 'Any Star Count';
      else selectedRating.value = data.filters.rating ? parseInt(data.filters.rating) : null;

      prompt.value = data.filters.prompt ? data.filters.prompt.join(', ') : '';
    } else {
      selectedModels.value = [];
      selectedLoras.value = [];
      selectedSamplers.value = [];
      selectedRating.value = null;
      prompt.value = '';
    }

    isEditing.value = true;
    displayCreateDialog.value = true;
  } catch (error) {
    console.error('Error fetching collection details:', error);
  }
};

const saveCollection = async () => {
  if (!newCollectionName.value.trim()) {
    toast.add({ severity: 'warn', summary: 'Validation Error', detail: 'Collection name cannot be empty.', life: 3000 });
    return;
  }

  const collectionData = {
    name: newCollectionName.value,
    isSmart: isSmartCollection.value,
    filters: isSmartCollection.value ? {
      models: selectedModels.value,
      loras: selectedLoras.value,
      samplers: selectedSamplers.value,
      rating: selectedRating.value !== null ? String(selectedRating.value) : null,
      prompt: prompt.value.split(',').map(p => p.trim()).filter(Boolean),
    } : null,
  };

  try {
    if (isEditing.value) {
      await api.put(`/collections/${encodeURIComponent(originalCollectionName.value)}`, collectionData);
      toast.add({ severity: 'success', summary: 'Success', detail: 'Collection updated successfully.', life: 3000 });
    } else {
      await api.post('/collections', collectionData);
      toast.add({ severity: 'success', summary: 'Success', detail: 'Collection created successfully.', life: 3000 });
    }

    displayCreateDialog.value = false;
    resetForm();
    await fetchCollections();
    store.triggerNavRefresh();
  } catch (error) {
    console.error('Error saving collection:', error);
  }
};

const fetchCollections = async () => {
  try {
    const response = await api.get('/collections');
    collections.value = response.data;
  } catch (error) {
    console.error('Error fetching collections:', error);
  }
};

const navigateToCollection = (name) => {
  store.setActiveCollection(name);
  router.push('/');
};

const onCardContextMenu = (event, collectionName) => {
  contextMenuSelection.value = collectionName;
  menuModel.value = [
    {
      label: 'Edit Collection',
      icon: 'pi pi-pencil',
      command: () => editCollection(collectionName)
    },
    {
      label: 'Delete Collection',
      icon: 'pi pi-trash',
      command: () => confirmDeleteCollection(collectionName)
    }
  ];
  if (cm.value) cm.value.show(event);
};

const confirmDeleteCollection = async (name) => {
  try {
    await api.delete(`/collections/${encodeURIComponent(name)}`);
    toast.add({ severity: 'success', summary: 'Deleted', detail: `Collection "${name}" removed.`, life: 3000 });
    await fetchCollections();
    store.triggerNavRefresh();
  } catch (e) {
    console.error('Failed to delete collection', e);
  }
};

const getThumbnailUrl = (path) => {
  return authenticatedUrl(`/api/images/thumbnail?path=${encodeURIComponent(path)}&size=300`);
};

onMounted(() => {
  fetchCollections();
  if (store.availableModels.length === 0) {
    store.loadFilters();
  }
  if (store.collectionToEdit) {
    editCollection(store.collectionToEdit);
    store.collectionToEdit = null;
  }
});
</script>

<template>
  <div class="flex h-full overflow-hidden collections-container-ds">
    <div class="flex-grow-1 flex flex-column overflow-y-auto collections-view p-4">
      <div class="flex flex-column align-items-center mb-5">
        <h1 class="view-title-hero mb-2">Collections</h1>
        <LButton variant="primary" size="md" class="mt-2" @click="openCreateDialog">
          <template #icon><Plus :size="16" /></template>
          Create New Collection
        </LButton>
      </div>

      <div class="grid px-4 justify-content-center">
        <div v-for="col in collections" :key="col.name" class="col-12 sm:col-6 md:col-4 lg:col-3 xl:col-2 p-3">
          <div
            class="collection-card-ds cursor-pointer"
            @click="navigateToCollection(col.name)"
            @contextmenu.prevent.stop="onCardContextMenu($event, col.name)"
          >
            <div class="stack-container mb-3">
              <div v-if="col.previewPaths && col.previewPaths.length > 0" class="stack">
                <div
                  v-for="(path, index) in col.previewPaths.slice().reverse()"
                  :key="path"
                  class="stack-item"
                  :style="{
                    transform: `translate(${index * 8}px, ${index * -8}px) scale(${1 - (index * 0.04)})`,
                    zIndex: 10 - index,
                    opacity: 1 - (index * 0.15)
                  }"
                >
                  <img :src="getThumbnailUrl(path)" alt="Preview" class="stack-img" />
                </div>
              </div>
              <div v-else class="stack-empty flex align-items-center justify-content-center">
                <FolderOpen :size="36" class="text-secondary opacity-50" />
              </div>
            </div>

            <div class="collection-info text-center">
              <div class="text-base font-bold text-white mb-1 text-overflow-ellipsis overflow-hidden white-space-nowrap px-2" :title="col.name">
                {{ col.name }}
              </div>
              <div class="flex align-items-center justify-content-center gap-1">
                <component :is="col.isSmart ? Bolt : Folder" :size="12" :class="col.isSmart ? 'text-accent' : 'text-secondary'" />
                <span class="text-xs uppercase tracking-wider font-semibold" :class="col.isSmart ? 'text-accent' : 'text-secondary'">
                  {{ col.isSmart ? 'Smart' : 'Static' }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Create / Edit Collection Dialog -->
      <Dialog
        :header="isEditing ? 'Edit Collection' : 'Create New Collection'"
        v-model:visible="displayCreateDialog"
        :modal="true"
        :style="{ width: '560px' }"
        class="ds-modal-wrapper"
      >
        <div class="flex flex-column gap-4 py-2">
          <div>
            <label for="collectionName" class="field-label-ds block mb-2">Collection Name</label>
            <InputText
              id="collectionName"
              v-model="newCollectionName"
              class="w-full ds-input"
              placeholder="e.g. Favorite Portrayals"
            />
          </div>

          <div>
            <div class="flex align-items-center gap-3">
              <LSwitch id="isSmartCollection" v-model="isSmartCollection">
                <span class="font-semibold text-sm select-none" :class="isSmartCollection ? 'text-accent' : 'text-white'">
                  Dynamic Auto-Population
                </span>
              </LSwitch>
            </div>
            <p class="text-xs text-secondary mt-2 mb-0">
              When enabled, this collection automatically includes images matching the smart filters below.
            </p>
          </div>

          <div v-if="isSmartCollection" class="flex flex-column gap-3 pt-2 border-top-1 border-white-alpha-10">
            <span class="section-title-ds">Smart Filters</span>

            <div class="grid grid-nogutter gap-3">
              <div class="col-12 md:col-6">
                <label class="field-label-ds block mb-1">Models</label>
                <MultiSelect
                  v-model="selectedModels"
                  :options="store.availableModels"
                  placeholder="Select Models"
                  class="w-full ds-input"
                  :scrollHeight="'30vh'"
                  @before-show="refreshFilters"
                />
              </div>

              <div class="col-12 md:col-6">
                <label class="field-label-ds block mb-1">Samplers</label>
                <MultiSelect
                  v-model="selectedSamplers"
                  :options="store.availableSamplers"
                  placeholder="Select Samplers"
                  class="w-full ds-input"
                  :scrollHeight="'30vh'"
                  @before-show="refreshFilters"
                />
              </div>

              <div class="col-12 md:col-6">
                <label class="field-label-ds block mb-1">LoRAs</label>
                <MultiSelect
                  v-model="selectedLoras"
                  :options="store.availableLoras"
                  placeholder="Select LoRAs"
                  class="w-full ds-input"
                  :scrollHeight="'30vh'"
                  @before-show="refreshFilters"
                />
              </div>

              <div class="col-12 md:col-6">
                <label class="field-label-ds block mb-1">Rating</label>
                <Dropdown
                  v-model="selectedRating"
                  :options="ratingOptions"
                  optionLabel="label"
                  optionValue="value"
                  placeholder="Select Rating"
                  class="w-full ds-input"
                  :scrollHeight="'30vh'"
                />
              </div>

              <div class="col-12">
                <label for="prompt" class="field-label-ds block mb-1">Prompt Contains</label>
                <InputText
                  id="prompt"
                  v-model="prompt"
                  class="w-full ds-input"
                  placeholder="e.g. portrait, 8k, masterpiece"
                />
              </div>
            </div>
          </div>
        </div>

        <template #footer>
          <div class="flex justify-content-end gap-2 pt-2">
            <LButton variant="secondary" size="sm" @click="displayCreateDialog = false">
              <template #icon><X :size="14" /></template>
              Cancel
            </LButton>
            <LButton variant="primary" size="sm" @click="saveCollection">
              <template #icon><Check :size="14" /></template>
              {{ isEditing ? 'Save Changes' : 'Create Collection' }}
            </LButton>
          </div>
        </template>
      </Dialog>

      <CustomContextMenu ref="cm" :model="menuModel" />
    </div>
  </div>
</template>

<style scoped>
.collections-container-ds {
  background: var(--color-bg-canvas, #0A0A0D);
}

.collection-card-ds {
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
  max-width: 220px;
  margin: 0 auto;
}

.collection-card-ds:hover {
  transform: translateY(-6px);
}

.stack-container {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.stack {
  position: relative;
  width: 160px;
  height: 160px;
}

.stack-item {
  position: absolute;
  inset: 0;
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  background: var(--color-surface-1, #14151B);
  box-shadow: var(--shadow-card, 0 8px 20px rgba(0, 0, 0, 0.4));
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.collection-card-ds:hover .stack-item {
  border-color: var(--color-accent-primary, #67E0D8);
}

.stack-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.stack-empty {
  width: 160px;
  height: 160px;
  background: var(--color-surface-1, #14151B);
  border: 2px dashed var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-md, 8px);
}

.field-label-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-secondary, #9294A3);
}

.section-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 700;
  color: var(--color-text-primary, #F2F3F7);
}

.ds-input {
  background: var(--color-surface-2, #23252F) !important;
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06)) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
  border-radius: var(--radius-sm, 6px) !important;
  font-family: var(--font-sans, Inter, sans-serif) !important;
  font-size: 13px !important;
}

.text-accent {
  color: var(--color-accent-primary, #67E0D8) !important;
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3) !important;
}

.text-white {
  color: var(--color-text-primary, #F2F3F7) !important;
}
</style>