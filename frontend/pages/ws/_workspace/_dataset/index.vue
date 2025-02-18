<!--
  - coding=utf-8
  - Copyright 2021-present, the Recognai S.L. team.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <re-loading v-if="$fetchState.pending" />
  <error
    v-else-if="$fetchState.error"
    :link="errorLink"
    :where="datasetName"
    :error="$fetchState.error"
  ></error>
  <task-search v-else :dataset="dataset" />
</template>

<script>
import { mapActions, mapGetters } from "vuex";
import { currentWorkspace, workspaceHome } from "@/models/Workspace";

export default {
  layout: "app",
  async fetch() {
    await this.fetchByName(this.datasetName);
  },
  computed: {
    ...mapGetters({
      findByName: "entities/datasets/findByName",
    }),

    dataset() {
      // This computed data makes that store updates could be shown here
      return this.findByName(this.datasetName);
    },
    datasetName() {
      return this.$route.params.dataset;
    },
    workspace() {
      return currentWorkspace(this.$route);
    },
    errorLink() {
      return workspaceHome(this.workspace);
    },
  },
  methods: {
    ...mapActions({
      fetchByName: "entities/datasets/fetchByName",
    }),
  },
};
</script>
