<template>
  <div>
    <v-dialog
      v-model="dialog"
      max-width="500"
      :disabled="waiting"
      :persistent="waiting"
      @click:outside="
        () => {
          form = {};
          this.$refs.addSemestre.reset();
          if (this.update == true) this.cancelUpdate();
          errorMessages.code = null;
        }
      "
    >
      <v-btn
        slot="activator"
        slot-scope="props"
        v-on="props.on"
        color="var(--primary-color)"
        style="color: white"
      >
        <v-icon left> mdi-plus </v-icon> Adicionar
      </v-btn>
      <v-card>
        <v-toolbar color="var(--primary-dark-color)" style="color: white"
          ><v-progress-linear
            v-if="waiting == true"
            indeterminate
          ></v-progress-linear>
          <h5>
            {{ update == true ? "Editar semestre" : "Adicionar novo semestre" }}
          </h5></v-toolbar
        >
        <v-card-text class="pt-6">
          <v-form v-model="validForm" ref="addSemestre">
            <v-text-field
              v-model="form.code"
              @keyup="form.code = $event.target.value.toUpperCase()"
              @keyup.enter="handleSubmit"
              :rules="codeRules"
              label="Código do semestre"
              required
              :disabled="update"
              :error="errorMessages.code != null"
              :error-messages="errorMessages.code"
            ></v-text-field>
            <v-menu
              v-model="menu"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="form.start_date"
                  label="Data de início do semestre"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                  :rules="dateRules"
                  @keyup.enter="handleSubmit"
                  :disabled="update"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="form.start_date"
                @input="
                  () => {
                    menu = false;
                    form.end_date = null;
                  }
                "
                :min="currentDate"
                locale="pt-br"
                no-title
                scrollable
              ></v-date-picker>
            </v-menu>
            <v-menu
              v-model="menu2"
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="form.end_date"
                  label="Data de término do semestre"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                  :rules="dateRules"
                  @keyup.enter="
                    update == true ? handleUpdate() : handleSubmit()
                  "
                ></v-text-field>
              </template>
              <v-date-picker
                locale="pt-br"
                no-title
                scrollable
                v-model="form.end_date"
                @input="
                  () => {
                    menu2 = false;
                  }
                "
                :min="minEndDate"
                :disabled="form.start_date == null || form.start_date == ''"
              ></v-date-picker>
            </v-menu>
          </v-form>
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn
            text
            color="red darken-1"
            @click="
              () => {
                dialog = false;
                this.$refs.addSemestre.reset();
                if (update == true) this.cancelUpdate();
                errorMessages.code = null;
              }
            "
            >Cancelar</v-btn
          >
          <v-btn
            text
            color="light-blue darken-4"
            @click.prevent="update == true ? handleUpdate() : handleSubmit()"
          >
            {{ update == true ? "Confirmar edição" : "Adicionar" }}</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
    <Snackbar
      type="success"
      :text="snackText"
      :show="stored"
      @hide="stored = false"
    />
    <Snackbar
      type="success"
      :text="snackText"
      :show="updated"
      @hide="updated = false"
    />
  </div>
</template>
<script>
import Snackbar from "../Snackbar";
import semestreService from "../../services/semestreService";
export default {
  components: {
    Snackbar,
  },
  props: ["dataToUpdate", "waiting"],
  data() {
    return {
      form: { code: "", start_date: "", end_date: "" },
      dialog: false,
      validForm: undefined,
      codeRules: [
        (v) => !!v || "Código da Disciplina é um campo obrigatório",
        (v) =>
          /\b[2][0][0-9]{2}[.][1-3]\b/g.test(v) ||
          "Código do Semestre deve seguir o formato NNNN.N",
      ],
      dateRules: [(v) => !!v || "Campo de Data Obrigatório"],
      stored: false,
      update: false,
      updated: false,
      menu: false,
      menu2: false,
      errorMessages: { code: null },
    };
  },
  methods: {
    async handleSubmit() {
      if (this.form.code) this.form.code.trim();
      if (this.form.start_date) this.form.start_date.trim();
      if (this.form.end_date) this.form.end_date.trim();
      try {
        if (this.$refs.addSemestre.validate()) {
          var semestre = await semestreService.store(this.form);
          this.dialog = false;
          this.$emit("handleSubmit", semestre.data);
          this.stored = true;
          this.form = {};
          this.$refs.addSemestre.reset();
          this.errorMessages.code = null;
        }
      } catch (error) {
        error.response.data.message.forEach((item) => {
          this.handleError(item);
        });
      }
    },
    async handleUpdate() {
      try {
        if (this.$refs.addSemestre.validate()) {
          var updatedData = await semestreService.update(this.form);
          this.$emit("handleUpdate", updatedData.data.semestre);
          this.dialog = false;
          this.updated = true;
          this.update = false;
          this.form = {};
          this.$refs.addSemestre.reset();
        }
      } catch (error) {
        console.log(error);
      }
    },
    handleError(error) {
      switch (error.field) {
        case "code": {
          this.errorMessages.code = error.message;
          break;
        }
      }
    },
    //new
    getOne() {
      semestreService.getOne(this.dataToUpdate).then((response) => {
        this.form = response.data;
      });
      this.dialog = true;
      this.update = true;
    },
    //new
    cancelUpdate() {
      this.update = false;
      this.$emit("cancelUpdate");
    },
  },
  watch: {
    dataToUpdate() {
      if (this.dataToUpdate != null) this.getOne();
    },
  },
  computed: {
    currentDate() {
      var today = new Date();
      var dd = String(today.getDate()).padStart(2, "0");
      var mm = String(today.getMonth() + 1).padStart(2, "0"); //January is 0!
      var yyyy = today.getFullYear();
      today = yyyy + "-" + mm + "-" + dd;
      return today;
    },
    // Fazer tratamento de mês
    minEndDate() {
      if (this.form.start_date) {
        var min = this.form.start_date;
        min = min.split("-");
        min[2] = parseInt(min[2]) + 1;
        if (min[2] < 10) {
          min[2] = `0${min[2]}`;
        }
        min = min[0] + "-" + min[1] + "-" + min[2];
        console.log(min);
        return min;
      } else {
        return this.currentDate;
      }
    },
    snackText() {
      return this.updated == false
        ? "Semestre Adicionado com Sucesso!"
        : "Semestre Atualizado com Sucesso!";
    },
  },
};
</script>
