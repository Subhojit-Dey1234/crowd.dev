<template>
  <app-page-wrapper size="full-width">
    <div class="member-list-page">
      <div class="mb-10">
        <div class="flex items-center justify-between">
          <div class="flex items-center">
            <h4>Organizations</h4>
          </div>
          <div class="flex items-center">
            <router-link
              v-if="hasPermissionToCreate"
              :to="{
                name: 'organizationCreate',
              }"
              :class="{
                'pointer-events-none cursor-not-allowed':
                  isCreateLockedForSampleData,
              }"
            >
              <el-button
                class="btn btn--primary btn--md"
                :disabled="isCreateLockedForSampleData"
              >
                Add organization
              </el-button>
            </router-link>
          </div>
        </div>
        <div class="text-xs text-gray-500">
          Overview of all organizations that relate to your
          community
        </div>
      </div>

      <cr-saved-views
        v-model="filters"
        :config="organizationSavedViews"
        :views="organizationViews"
        @update:model-value="organizationFilter.alignFilterList($event)"
      />
      <cr-filter
        ref="organizationFilter"
        v-model="filters"
        :config="organizationFilters"
        :search-config="organizationSearchFilter"
        :saved-views-config="organizationSavedViews"
        @fetch="fetch($event)"
      />
      <app-organization-list-table
        :has-organizations="totalOrganizations > 0"
        :is-page-loading="loading"
      />
    </div>
  </app-page-wrapper>
</template>

<script setup lang="ts">
import { computed, ref, onMounted } from 'vue';
import AppPageWrapper from '@/shared/layout/page-wrapper.vue';
import AppOrganizationListTable from '@/modules/organization/components/list/organization-list-table.vue';
import {
  mapGetters,
} from '@/shared/vuex/vuex.helpers';
import CrSavedViews from '@/shared/modules/saved-views/components/SavedViews.vue';
import CrFilter from '@/shared/modules/filters/components/Filter.vue';
import { useOrganizationStore } from '@/modules/organization/store/pinia';
import { storeToRefs } from 'pinia';
import { organizationFilters, organizationSearchFilter } from '@/modules/organization/config/filters/main';
import { organizationSavedViews, organizationViews } from '@/modules/organization/config/saved-views/main';
import { FilterQuery } from '@/shared/modules/filters/types/FilterQuery';
import { OrganizationService } from '@/modules/organization/organization-service';
import { OrganizationPermissions } from '../organization-permissions';

const { currentUser, currentTenant } = mapGetters('auth');

const organizationStore = useOrganizationStore();
const { filters, totalOrganizations, savedFilterBody } = storeToRefs(organizationStore);
const { fetchOrganizations } = organizationStore;

const loading = ref(true);
const organizationCount = ref(0);

const organizationFilter = ref<CrFilter | null>(null);

const hasPermissionToCreate = computed(
  () => new OrganizationPermissions(
    currentTenant.value,
    currentUser.value,
  ).create,
);
const isCreateLockedForSampleData = computed(
  () => new OrganizationPermissions(
    currentTenant.value,
    currentUser.value,
  ).createLockedForSampleData,
);

const doGetOrganizationCount = () => {
  (OrganizationService.listOrganizations({
    limit: 1,
    offset: 0,
  }) as Promise<any>)
    .then(({ count }) => {
      organizationCount.value = count;
    });
};

const showLoading = (filter: any, body: any): boolean => {
  const saved: any = { ...savedFilterBody.value };
  delete saved.offset;
  delete saved.limit;
  delete saved.orderBy;
  const compare = {
    ...body,
    filter,
  };
  return JSON.stringify(saved) !== JSON.stringify(compare);
};

const fetch = ({
  filter, offset, limit, orderBy, body,
}: FilterQuery) => {
  if (!loading.value) {
    loading.value = showLoading(filter, body);
  }
  fetchOrganizations({
    body: {
      ...body,
      filter,
      offset,
      limit,
      orderBy,
    },
  })
    .finally(() => {
      loading.value = false;
    });
};

onMounted(async () => {
  doGetOrganizationCount();
  (window as any).analytics.page('Organization');
});
</script>
