<script setup>
import { ref } from 'vue';
import {
  Dialog,
  DialogPanel,
  TransitionChild,
  TransitionRoot,
} from '@headlessui/vue';
import { XMarkIcon } from '@heroicons/vue/24/outline';

const mobileFiltersOpen = ref(false);
</script>
<template>
  <div class="bg-white">
    <div>
      <!-- Mobile filter dialog -->
      <TransitionRoot
        as="template"
        :show="mobileFiltersOpen"
      >
        <Dialog
          as="div"
          class="relative z-40 lg:hidden"
          @close="mobileFiltersOpen = false"
        >
          <TransitionChild
            as="template"
            enter="transition-opacity ease-linear duration-300"
            enter-from="opacity-0"
            enter-to="opacity-100"
            leave="transition-opacity ease-linear duration-300"
            leave-from="opacity-100"
            leave-to="opacity-0"
          >
            <div class="fixed inset-0 bg-black bg-opacity-25" />
          </TransitionChild>

          <div class="fixed inset-0 z-40 flex">
            <TransitionChild
              as="template"
              enter="transition ease-in-out duration-300 transform"
              enter-from="translate-x-full"
              enter-to="translate-x-0"
              leave="transition ease-in-out duration-300 transform"
              leave-from="translate-x-0"
              leave-to="translate-x-full"
            >
              <DialogPanel class="relative ml-auto flex h-full w-full max-w-sm flex-col bg-white pt-4 shadow-xl">
                <div class="flex items-center justify-between px-4">
                  <h2 class="text-lg font-bold text-gray-500 uppercase">
                    Filtres
                  </h2>
                  <button
                    type="button"
                    class="-mr-2 flex h-10 w-10 items-center justify-center p-2 text-gray-400 hover:text-gray-500"
                    @click="mobileFiltersOpen = false"
                  >
                    <span class="sr-only">Fermer menu</span>
                    <XMarkIcon
                      class="h-6 w-6"
                      aria-hidden="true"
                    />
                  </button>
                </div>
                <div class="flex-1 overflow-y-auto">
                  <div class="pb-4 pt-4 px-4 text-sm font-bold">
                    {{ isRandomView ? ecoles.length + ' écoles (aléatoire)' : ecoles.length + ' écoles trouvées' }}
                  </div>

                  <!-- Search by name -->
                  <div class="border-t border-gray-200 pb-4 pt-4 px-4">
                    <label
                      for="search-name-mobile"
                      class="block text-sm font-bold text-gray-900 uppercase"
                    >
                      Recherche par nom
                    </label>
                    <input
                      id="search-name-mobile"
                      v-model="search_text"
                      type="text"
                      placeholder="Nom de l'école"
                      class="mt-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-age focus:ring-age sm:text-sm"
                      @input="searchByName"
                    >
                  </div>

                  <!-- Search by postal code -->
                  <div class="border-t border-gray-200 pb-4 pt-4 px-4">
                    <label
                      for="search-postal-mobile"
                      class="block text-sm font-bold text-gray-900 uppercase"
                    >
                      Code postal
                    </label>
                    <input
                      id="search-postal-mobile"
                      v-model="search_postal"
                      type="number"
                      placeholder="ex: 1000"
                      class="mt-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-age focus:ring-age sm:text-sm"
                      @input="searchByPostalCode"
                    >
                  </div>

                  <!-- Filters -->
                  <div
                    v-for="section in filters"
                    :key="section.name"
                    class="border-t border-gray-200 pb-4 pt-4"
                  >
                    <fieldset>
                      <legend class="w-full px-2">
                        <button
                          class="flex w-full items-center justify-between p-2 text-gray-400 hover:text-gray-500"
                          @click="section.collapsed = !section.collapsed"
                        >
                          <span class="text-sm font-bold text-gray-900 uppercase">{{ section.name }}
                            <span
                              v-if="active_filters[section.id]"
                              class="text-xs font-normal text-age"
                            >(1)</span>
                          </span>
                          <ChevronDownIcon
                            v-if="!section.collapsed"
                            class="h-5 w-5"
                          />
                          <ChevronRightIcon
                            v-else
                            class="h-5 w-5"
                          />
                        </button>
                      </legend>
                      <div
                        v-show="!section.collapsed && data_loaded"
                        class="px-4 pb-2"
                      >
                        <div class="space-y-3">
                          <button
                            v-for="option in section.options"
                            :key="option"
                            class="flex items-center text-left"
                            :class="active_filters[section.id] === option ? 'font-bold text-age' : ''"
                            @click="filterCategory(section.id, option)"
                          >
                            {{ option }} <span class="text-gray-500 ml-1">({{ getCountCategory(section.id, option) }})</span>
                          </button>
                        </div>
                      </div>
                    </fieldset>
                  </div>

                  <div class="px-4 pt-4">
                    <button
                      v-if="Object.keys(active_filters).length > 0 || search_text || search_postal"
                      class="w-full rounded-md bg-age px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-blue-500"
                      @click="resetFilter"
                    >
                      Réinitialiser les filtres
                    </button>
                  </div>
                </div>
                <div class="border-t border-gray-200 p-4">
                  <button
                    class="w-full rounded-md bg-age px-4 py-3 text-sm font-semibold text-white shadow-sm hover:bg-blue-500"
                    @click="mobileFiltersOpen = false"
                  >
                    Voir les résultats ({{ ecoles.length }})
                  </button>
                </div>
              </DialogPanel>
            </TransitionChild>
          </div>
        </Dialog>
      </TransitionRoot>

      <main class="mx-auto max-w-2xl px-4 py-4 sm:px-6 sm:py-6 lg:max-w-7xl lg:px-8 lg:py-8">
        <div class="border-b border-gray-200 pb-4 lg:pb-10">
          <!-- logo -->
          <div class="flex items-center justify-center">
            <img
              src="https://raw.githubusercontent.com/data-cfwb/charte-graphique/main/pastilles_PNG_et_SVG_24px/pastille_ENS24.svg"
              alt="Fédération Wallonie-Bruxelles"
              width="48"
              class="px-1 sm:px-1"
            >
            <h1 class="text-2xl sm:text-3xl lg:text-4xl font-bold tracking-tight text-gray-900">
              Les écoles en FWB
            </h1>
          </div>
          <!-- hide on small -->
          <p class="mt-2 text-sm sm:text-base text-gray-500 text-center">
            Trouvez des informations sur les <span class="font-semibold">{{ original_ecoles.length }}</span> établissements d'enseignement en Fédération Wallonie-Bruxelles.
          </p>

          <!-- Sticky mobile toolbar -->
          <div class="sticky top-0 z-30 -mx-4 px-4 py-3 bg-white/95 backdrop-blur-sm border-b border-gray-200 lg:hidden">
            <div class="flex items-center justify-between">
              <span class="text-sm font-medium text-gray-700">
                {{ isRandomView ? 'Aléatoire' : ecoles.length + ' écoles' }}
              </span>
              <div class="flex items-center gap-2">
                <button
                  type="button"
                  class="inline-flex items-center gap-1.5 rounded-full bg-gray-100 px-3 py-1.5 text-sm font-medium text-gray-700 hover:bg-gray-200"
                  @click="mobileFiltersOpen = true"
                >
                  <FunnelIcon class="h-4 w-4" />
                  Filtres
                  <span
                    v-if="Object.keys(active_filters).length > 0"
                    class="inline-flex items-center justify-center rounded-full bg-age w-5 h-5 text-xs text-white"
                  >
                    {{ Object.keys(active_filters).length }}
                  </span>
                </button>
                <button
                  v-if="!seeMap"
                  type="button"
                  class="inline-flex items-center gap-1 rounded-full bg-green-100 px-3 py-1.5 text-sm font-medium text-green-700 hover:bg-green-200"
                  @click="seeMap = true"
                >
                  <MapPinIcon class="h-4 w-4" />
                  Carte
                </button>
                <button
                  v-if="seeMap"
                  type="button"
                  class="inline-flex items-center gap-1 rounded-full bg-blue-100 px-3 py-1.5 text-sm font-medium text-blue-700 hover:bg-blue-200"
                  @click="seeMap = false"
                >
                  <ListBulletIcon class="h-4 w-4" />
                  Liste
                </button>
              </div>
            </div>
          </div>

          <div class="pt-6 lg:pt-10 lg:grid lg:grid-cols-3 lg:gap-x-8 xl:grid-cols-4">
            <aside>
              <h2 class="sr-only">
                Filtres
              </h2>

              <div class="hidden lg:block">
                <!-- Search by name -->
                <div class="py-6">
                  <label
                    for="search-name"
                    class="block text-md font-bold leading-6 text-gray-900 uppercase"
                  >
                    <MagnifyingGlassIcon class="h-6 w-6 text-age inline-flex items-baseline" />
                    Recherche par nom
                  </label>
                  <input
                    id="search-name"
                    v-model="search_text"
                    type="text"
                    placeholder="Nom de l'école"
                    class="mt-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-age focus:ring-age sm:text-sm"
                    @input="searchByName"
                  >
                </div>

                <!-- Search by postal code -->
                <div class="py-6 pt-4">
                  <label
                    for="search-postal"
                    class="block text-md font-bold leading-6 text-gray-900 uppercase"
                  >
                    <MapPinIcon class="h-6 w-6 text-age inline-flex items-baseline" />
                    Code postal
                  </label>
                  <input
                    id="search-postal"
                    v-model="search_postal"
                    type="number"
                    placeholder="ex: 1000"
                    class="mt-2 block w-full rounded-md border-gray-300 shadow-sm focus:border-age focus:ring-age sm:text-sm"
                    @input="searchByPostalCode"
                  >
                </div>

                <div
                  v-if="Object.keys(active_filters).length > 0 || search_text || search_postal"
                  class="pt-4"
                >
                  <button
                    class="w-full rounded-md bg-age px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-blue-500"
                    @click="resetFilter"
                  >
                    Réinitialiser les filtres
                  </button>
                </div>

                <div
                  v-for="section in filters"
                  :key="section.name"
                  class="pt-4"
                >
                  <fieldset>
                    <legend class="w-full">
                      <button
                        class="flex w-full items-center justify-between text-md leading-6 font-bold text-gray-900 uppercase hover:text-age"
                        @click="section.collapsed = !section.collapsed"
                      >
                        <span>{{ section.name }}
                          <span
                            v-if="active_filters[section.id]"
                            class="text-xs font-normal text-age"
                          >(1)</span>
                        </span>
                        <ChevronDownIcon
                          v-if="!section.collapsed"
                          class="h-5 w-5"
                        />
                        <ChevronRightIcon
                          v-else
                          class="h-5 w-5"
                        />
                      </button>
                    </legend>
                    <div
                      v-show="!section.collapsed"
                      class="space-y-3 pt-2 max-h-64 overflow-y-auto"
                    >
                      <button
                        v-for="option in section.options"
                        :key="option"
                        class="flex items-center hover:text-age text-left"
                        :class="active_filters[section.id] === option ? 'font-bold text-age' : ''"
                        @click="filterCategory(section.id, option)"
                      >
                        {{ option }} <span class="text-gray-500 ml-1">({{ getCountCategory(section.id, option) }})</span>
                      </button>
                    </div>
                  </fieldset>
                </div>
              </div>
            </aside>

            <!-- Ecole content -->
            <div class="mt-6 lg:col-span-2 lg:mt-0 xl:col-span-3">
              <!-- Your content -->
              <div>
                <LoadingFwb v-if="!data_loaded" />
                <div v-else>
                  <div class="hidden lg:flex lg:items-center">
                    <div class="flex-auto">
                      <p class="text-sm text-gray-700">
                        {{ isRandomView ? ecoles.length + ' écoles (aléatoire)' : ecoles.length + ' écoles trouvées' }}
                      </p>
                    </div>
                    <div class="ml-16 flex-none">
                      <button
                        v-if="!seeMap"
                        type="button"
                        class="relative inline-flex items-center rounded-md bg-green-600 px-3 py-2 text-center text-sm font-semibold text-white shadow-sm hover:bg-green-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-green-600"
                        @click="seeMap = !seeMap"
                      >
                        <MapPinIcon class="h-6 w-6 text-white" />
                        Voir sur une carte
                      </button>
                      <button
                        v-if="seeMap"
                        type="button"
                        class="relative inline-flex items-center rounded-md bg-blue-600 px-3 py-2 text-center text-sm font-semibold text-white shadow-sm hover:bg-blue-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600"
                        @click="seeMap = !seeMap"
                      >
                        <ListBulletIcon class="h-6 w-6 text-white" />
                        Liste des écoles
                      </button>
                    </div>
                  </div>
                  <div class="mt-6">
                    <div
                      v-if="seeMap"
                      class="overflow-hidden rounded-lg"
                    >
                      <MapComponent
                        :ecoles="allEcolesForMap"
                        :selected-ecole="selected_ecole"
                        @select-ecole="selectEcole"
                      />
                    </div>
                    <ListEcoles
                      v-if="!seeMap"
                      :ecoles="ecoles"
                      @select-ecole="selectEcole"
                    />
                    <EcoleDescription
                      v-if="selected_ecole && selected_ecole.nom"
                      :ecole="selected_ecole"
                      @close="closeEcole"
                      @filter="applyFilterFromEcole"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>
    <FooterModule />
  </div>
</template>

<script>
import MapComponent from '@/components/MapComponent.vue';
import FooterModule from '@/components/partials/FooterModule.vue';
import ListEcoles from '@/components/ListEcoles.vue';
import EcoleDescription from '@/components/partials/EcoleDescription.vue';
import { MapPinIcon, ListBulletIcon, MagnifyingGlassIcon, ChevronDownIcon, ChevronRightIcon, FunnelIcon } from '@heroicons/vue/24/outline';

export default {
  components: {
    MapComponent,
    ListEcoles,
    EcoleDescription,
    MapPinIcon,
    ListBulletIcon,
    MagnifyingGlassIcon,
    ChevronDownIcon,
    ChevronRightIcon,
    FunnelIcon,
    FooterModule
  },
  data() {
    return {
      title: 'Les écoles en FWB',
      searchDebounceTimer: null,
      filters: [
        {
          id: 'type',
          name: 'Type d\'enseignement',
          options: [],
          collapsed: true,
        },
        {
          id: 'reseau',
          name: 'Réseau',
          options: [],
          collapsed: true,
        },
        {
          id: 'arr',
          name: 'Arrondissement',
          options: [],
          collapsed: true,
        },
        {
          id: 'commune_etab',
          name: 'Commune',
          options: [],
          collapsed: true,
        }
      ],
      selected_ecole: null,
      original_ecoles: [],
      ecoles: [],
      active_filters: {},
      search_text: '',
      search_postal: '',
      data_loaded: false,
      seeMap: false,
      filterCounts: {},
      isRandomView: true,
    };
  },
  computed: {
    allEcolesForMap() {
      return this.isRandomView ? this.original_ecoles : this.ecoles;
    },
  },
  created() {
    this.getEcoles();
  },
  methods: {
    selectEcole(ecole) {
      this.selected_ecole = ecole;
      this.updateUrl();
    },
    closeEcole() {
      this.selected_ecole = null;
      this.updateUrl();
    },
    applyFilterFromEcole(category, value) {
      this.selected_ecole = null;
      this.active_filters = { [category]: value };
      this.applyFilters();
      this.updateUrl();
    },
    updateUrl() {
      const url = new URL(window.location);
      if (this.selected_ecole) {
        url.searchParams.set('etablissement', this.selected_ecole.id_etab);
        url.searchParams.set('implantation', this.selected_ecole.id_impl);
      } else {
        url.searchParams.delete('etablissement');
        url.searchParams.delete('implantation');
      }
      history.replaceState(null, '', url);
    },
    getCountCategory(category, item) {
      const counts = this.filterCounts[category];
      return counts ? (counts[item] || 0) : 0;
    },
    defineFilters() {
      const counts = {};
      for (let i = 0; i < this.filters.length; i++) {
        const id = this.filters[i].id;
        const valueCounts = {};
        this.original_ecoles.forEach(ecole => {
          const val = ecole[id];
          if (val && val.trim()) {
            valueCounts[val] = (valueCounts[val] || 0) + 1;
          }
        });
        counts[id] = valueCounts;
        this.filters[i].options = Object.keys(valueCounts).sort();
      }
      this.filterCounts = counts;
    },
    filterCategory(category, value) {
      if (this.active_filters[category] === value) {
        delete this.active_filters[category];
      } else {
        this.active_filters[category] = value;
      }
      this.applyFilters();
    },
    searchByName() {
      if (this.search_text && this.search_text.length < 3) return;
      this.debouncedApplyFilters();
    },
    searchByPostalCode() {
      if (this.search_postal && this.search_postal.toString().length < 2) return;
      this.debouncedApplyFilters();
    },
    debouncedApplyFilters() {
      clearTimeout(this.searchDebounceTimer);
      this.searchDebounceTimer = setTimeout(() => {
        this.applyFilters();
      }, 300);
    },
    applyFilters() {
      this.isRandomView = false;
      let filtered = [...this.original_ecoles];

      // Apply category filters
      Object.keys(this.active_filters).forEach(category => {
        const value = this.active_filters[category];
        filtered = filtered.filter(ecole => ecole[category] === value);
      });

      // Apply name search
      if (this.search_text) {
        const searchLower = this.search_text.toLowerCase();
        filtered = filtered.filter(ecole =>
          ecole.nom && ecole.nom.toLowerCase().includes(searchLower)
        );
      }

      // Apply postal code search
      if (this.search_postal) {
        filtered = filtered.filter(ecole =>
          ecole.cp_etab && ecole.cp_etab.toString().includes(this.search_postal)
        );
      }

      this.ecoles = filtered;
      this.orderByCommune();
    },
    orderByCommune() {
      this.ecoles.sort((a, b) => {
        if (!a.commune_etab) return 1;
        if (!b.commune_etab) return -1;
        return a.commune_etab.localeCompare(b.commune_etab);
      });
    },
    resetFilter() {
      this.active_filters = {};
      this.search_text = '';
      this.search_postal = '';
      this.isRandomView = true;
      this.ecoles = this.shuffleArray([...this.original_ecoles]);
    },
    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    prepareProperties(ecoles) {
      ecoles.forEach(ecole => {
        ecole.latLong = [parseFloat(ecole.lat), parseFloat(ecole.lng)];
        ecole.fullAddress = `${ecole.adr_etab || ''}, ${ecole.cp_etab || ''} ${ecole.loc_etab || ''}`.trim();
        if (ecole.nom) {
          ecole.nom = ecole.nom.replace(/\n/g, ' ').trim();
        }
        if (ecole.commune_etab) {
          ecole.commune_etab = ecole.commune_etab.replace(/\n/g, ' ').trim();
        }
        if (ecole.loc_etab) {
          ecole.loc_etab = ecole.loc_etab.replace(/\n/g, ' ').trim();
        }
      });
    },
    async getEcoles() {
      this.data_loaded = false;
      const response = await fetch(`${process.env.BASE_URL}data/ecoles.json`);
      this.original_ecoles = await response.json();
      this.prepareProperties(this.original_ecoles);
      this.defineFilters();
      this.ecoles = this.shuffleArray([...this.original_ecoles]);
      this.data_loaded = true;

      // Deep linking: open school from URL params
      const params = new URLSearchParams(window.location.search);
      const etab = params.get('etablissement');
      const impl = params.get('implantation');
      if (etab && impl) {
        const found = this.original_ecoles.find(e =>
          e.id_etab == etab && e.id_impl == impl
        );
        if (found) this.selected_ecole = found;
      }
    },
  },
};
</script>
