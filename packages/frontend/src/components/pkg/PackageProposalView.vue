<script>
import gql from 'graphql-tag'
import { useQuery, useResult } from '@vue/apollo-composable'
import { pkgProposalFragment } from './fragments'
import { computed } from '@vue/composition-api'
import { installationAvailable } from '@/util/installation'

import PackageViewLayout from './PackageViewLayout.vue'
import PackageProposalUpvoteButton from './PackageProposalUpvoteButton.vue'
import PackageProposalApproveButton from './PackageProposalApproveButton.vue'
import PackageProposalRemoveButton from './PackageProposalRemoveButton.vue'
import PackageInstallButton from './PackageInstallButton.vue'
import PackageReleaseCount from './PackageReleaseCount.vue'
import RouteTab from '../RouteTab.vue'

export default {
  components: {
    PackageViewLayout,
    PackageProposalApproveButton,
    PackageProposalRemoveButton,
    PackageProposalUpvoteButton,
    PackageReleaseCount,
    PackageInstallButton,
    RouteTab,
  },

  props: {
    packageId: {
      type: String,
      required: true,
    },

    projectTypeId: {
      type: String,
      required: true,
    },
  },

  setup (props, { root }) {
    const { result, loading } = useQuery(gql`
      query PackageProposal ($id: ID!) {
        pkg: packageProposal (id: $id) {
          ...pkgProposal
        }
      }
      ${pkgProposalFragment}
    `, () => ({
      id: props.packageId,
    }))
    const pkg = useResult(result)

    const { result: additionalResult } = useQuery(gql`
      query PackageProposalProjectTypes ($packageId: ID!) {
        pkg: packageProposal (id: $packageId) {
          id
          projectTypes {
            id
            inTeam
          }
        }
      }
    `, props)
    const inTeam = computed(() => {
      if (additionalResult.value) {
        return additionalResult.value.pkg.projectTypes.some(pt => pt.inTeam)
      }
    })

    return {
      pkg,
      loading,
      inTeam,
      installationAvailable,
    }
  },

  metaInfo () {
    if (!this.pkg) return

    return {
      title: `[Proposal] ${this.pkg.name}`,
    }
  },
}
</script>

<template>
  <PackageViewLayout
    :pkg="pkg"
    :loading="loading"
    :error="pkg && !pkg.repo ? 'No GitHub repository found' : null"
  >
    <template #actions>
      <template v-if="inTeam">
        <PackageInstallButton
          v-if="installationAvailable"
          :pkg="pkg"
          class="mr-4"
        />

        <PackageProposalApproveButton
          :project-type-id="projectTypeId"
          :proposal="pkg"
          class="px-8 py-4 mr-4"
        />

        <PackageProposalRemoveButton
          :project-type-id="projectTypeId"
          :proposal="pkg"
          class="px-8 py-4 mr-4"
        />
      </template>

      <PackageProposalUpvoteButton
        :pkg="pkg"
      />
    </template>

    <template #tabs>
      <RouteTab
        :to="{ name: 'package-proposal' }"
        class="flex-none"
        exact
      >
        General
      </RouteTab>

      <RouteTab
        :to="{ name: 'package-proposal-releases' }"
        class="flex-none"
      >
        Releases
        <PackageReleaseCount
          v-if="!loading"
          :pkg="pkg"
        />
      </RouteTab>

      <RouteTab
        :to="{ name: 'package-proposal-insight' }"
        class="flex-none"
      >
        Insight
      </RouteTab>

      <RouteTab
        :to="{ name: 'package-proposal-data-sources' }"
        class="flex-none"
      >
        Data sources
      </RouteTab>

      <RouteTab
        v-if="inTeam"
        :to="{ name: 'package-proposal-edit' }"
        class="flex-none"
      >
        Edit
      </RouteTab>
    </template>
  </PackageViewLayout>
</template>
