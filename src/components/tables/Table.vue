<template>
  <div>
    <v-data-table
      :loading="loading"
      :items="filteredList"
      :headers="headers"
      loading-text="Carregando... Por favor, aguarde"
      :items-per-page="10"
      class="elevation-1 row-pointer table"
      :no-data-text="notFound"
    >
      <template v-slot:item.action="{ item }">
        <v-btn
          @click="handleUpdate(item)"
          elevation="2"
          small
          color="warning"
          tile
        >
          <v-icon left> mdi-pencil </v-icon>Editar</v-btn
        >
        <v-btn
          @click="
            () => {
              confirmDialog = true;
              selectedItem = item;
            }
          "
          elevation="2"
          small
          color="error"
          tile
          class="ml-2"
          ><v-icon left> mdi-delete </v-icon>Remover</v-btn
        >
      </template>
      <template v-slot:item.actionTurma="{ item }">
        <v-btn @click="handleTutor(item)" elevation="2" small color="info" tile
          ><v-icon left> mdi-account-supervisor </v-icon>Tutores</v-btn
        >
        <v-btn
          @click="
            () => {
              confirmDialog = true;
              selectedItem = item;
            }
          "
          elevation="2"
          small
          color="error"
          tile
          class="ml-2"
          ><v-icon left> mdi-delete </v-icon>Remover</v-btn
        >
      </template>
    </v-data-table>
    <v-dialog
      v-model="confirmDialog"
      max-width="500"
      :disabled="waiting"
      :persistent="waiting"
      @keydown.enter="handleDelete"
    >
      <v-card>
        <v-toolbar color="var(--primary-dark-color)" style="color: white"
          ><h6>Deseja remover o item selecionado?</h6></v-toolbar
        >
        <v-progress-linear
          v-if="waiting == true"
          indeterminate
        ></v-progress-linear>
        <v-card-actions class="justify-end">
          <v-btn
            text
            color="red darken-1"
            @click="confirmDialog = false"
            :disabled="waiting"
            >Cancelar</v-btn
          >
          <v-btn
            text
            color="light-blue darken-4"
            @click="handleDelete"
            :disabled="waiting"
            >Remover</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
export default {
  props: ["loading", "headers", "filteredList", "notFound", "waiting"],
  data() {
    return {
      confirmDialog: false,
      selectedItem: undefined,
    };
  },
  methods: {
    handleDelete() {
      this.$emit("handleDelete", this.selectedItem);
    },
    handleUpdate(item) {
      this.$emit("handleUpdate", item);
    },
    handleTutor(item) {
      this.$emit("handleTutor", item);
    },
    getRows(rows) {
      const result = {};
      const _ = require("lodash");
      _.forEach(rows, (i, key) => {
        if (i.children) {
          _.forEach(i.children, (i1, key1) => {
            result["c" + key1] = i1;
          });
        } else result[key] = i;
      });
      console.log(result);
      return result;
    },
  },
  watch: {
    waiting(newValue) {
      if (newValue == false) this.confirmDialog = false;
    },
  },
};
</script>
<style>
td {
  word-break: break-word !important;
  overflow-wrap: break-word !important;
}
</style>
