<template>
  <v-container fluid class="places">
    <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details></v-text-field>
    <v-data-table
      dense
      :headers="headers"
      :items="items"
      :items-per-page="10"
      :item-class="itemClass"
      :search="search"
      @click:row="onClick"
    >
      <template v-slot:item.unmatch="{ item }">
        <v-icon medium dense v-if="item.matchedPage" @click.stop="unmatch(item)">mdi-link-variant-off</v-icon>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import { mapState } from "vuex";
import { HELPPAGES, PLACES } from "@/utils/ModelUtils";
import { fetchItem, loadPageTitle } from "@/utils/WeRelateUtils";

export default {
  name: "Places",
  created() {},
  mounted() {
    loadPageTitle(HELPPAGES[PLACES]);
  },
  data() {
    return {
      search: "",
      headers: [
        {
          text: "Place",
          value: "place"
        },
        {
          text: "Matched Page",
          value: "matchedPage"
        },
        {
          text: "Unmatch",
          value: "unmatch"
        }
      ]
    };
  },
  computed: {
    items() {
      if (!this.gedcom.model.places) {
        return [];
      }
      return this.gedcom.model.places.map(it => {
        return {
          place: it["@text"],
          matchedPage: it["@match"],
          cls: it["@read"] === "true" ? "place_read" : "place_unread",
          id: it["@id"]
        };
      });
    },
    ...mapState(["gedcom"])
  },
  methods: {
    itemClass(item) {
      return item.cls;
    },
    unmatch(item) {
      try {
        let place = this.gedcom.model.places.find(it => it["@id"] === item.id);
        this.$store.dispatch("gedcomUnmatch", { data: place });
      } catch (err) {
        this.$store.dispatch("notificationsAdd", err);
      }
    },
    onClick(item) {
      try {
        let place = this.gedcom.model.places.find(it => it["@id"] === item.id);
        this.$store.dispatch("prefsMarkRead", {
          model: this.gedcom.model,
          data: place,
          component: PLACES
        });
        fetchItem(this.gedcom.model, place, PLACES);
      } catch (err) {
        this.$store.dispatch("notificationsAdd", err);
      }
    }
  }
};
</script>

<style>
.places {
  margin-top: 0;
  padding-top: 0;
}
.place_read {
  cursor: pointer;
}
.place_unread {
  cursor: pointer;
  font-weight: bold;
}
</style>
