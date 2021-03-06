<template>
  <div class="container">
    <div class="row py-2 mb-4 bg-light rounded">
      <div class="col-12">
        <router-link
          to="/reports/"
          class="btn btn-primary btn-sm">
          <icon
            :icon="['fas', 'book']"
            class="mr-1"/>
          Create Reports
        </router-link>

        <button
          class="btn btn-secondary btn-sm float-right ml-2"
          @click.prevent
          @click="getEntries">
          <icon
            :icon="['fas', 'sync']"
            class="mr-1"/>
          Sync
        </button>
      </div>
    </div>

    <entry-modal
      v-if="modal.show && (this.$perms.add_entry || this.$perms.change_entry)"
      id="entry-modal"
      :config="modal"
      @close="toggleModal"/>

    <chart :entries="entries"/>

    <div
      v-if="this.$perms.view_entry"
      id="entry-rows">
      <div
        v-for="(entryBlock, blockIndex) in entries"
        :index="blockIndex"
        :key="entryBlock.id"
        class="mb-4">
        <div class="row inset-row">
          <div class="col-6">
            <h2 class="display-4 text-muted">
              {{ $moment(entryBlock.date).format('ddd[,] MMMM Do') }}
            </h2>
          </div>
          <div class="col-6">
            <h5 class="float-right">
              <span class="badge badge-success">
                {{ $moment.duration(entryBlock.duration, 'hours').format('h[h] m[m]') }}
              </span>
            </h5>
          </div>
        </div>
        <div class="rounded">
          <template v-if="entryBlock.entries.length > 0">
            <entry
              v-for="(entry, entryIndex) in entryBlock.entries"
              :entry="entry"
              :index="entryIndex"
              :key="entry.id"
              :editable="editable"
              :toggle-edit-modal="toggleModal"
              @delete-entry="deleteEntry(blockIndex, entryIndex)"/>
          </template>
          <template v-else>
            <div class="entry row py-3 bg-light small">
              <div class="col">
                <strong>No entries</strong> for this day.
              </div>
            </div>
          </template>
        </div>
      </div>

      <div class="row bg-success text-white py-2 mb-4 rounded">
        <div class="ml-auto col-sm-2 text-right">
          Subtotal<br>
          <strong>Total</strong>
        </div>
        <div class="col-sm-4">
          <icon
            :icon="['fas', 'clock']"
            class="mr-1"/>
          {{ $moment.duration(subtotal, 'hours').format('d[d] h[h] m[m]') }}<br>
          <icon
            :icon="['fas', 'clock']"
            class="mr-1"/>
          <strong>{{ $moment.duration(total, 'hours').format('d[d] h[h] m[m]') }}</strong>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
import {mapGetters, mapActions, mapState} from 'vuex';

import Datepicker from '../datepicker.vue';
import Entry from '../entry.vue';
import Pager from '../pager.vue';
import Select2 from '../select2.vue';
import EntryModal from './entry-modal.vue';
import Chart from './chart.vue';


export default {
  components: {
    Datepicker,
    Entry,
    Pager,
    Select2,
    EntryModal,
    Chart,
  },
  data() {
    return {
      editable: true,
      modal: {
        entry: null,
        show: false,
      },
    };
  },
  computed: {
    ...mapState({
      total: state => state.entries.total,
    }),
    ...mapGetters({
      entries: 'entries/getEntriesByDay',
      subtotal: 'entries/getEntriesByDayTotal',
    }),
  },
  methods: {
    ...mapActions('entries', [
      'getEntries',
    ]),
    toggleModal(entry) {
      if (entry) this.modal.entry = entry;
      else this.modal.entry = null;
      this.modal.show = !this.modal.show;
    },
  },
};
</script>
