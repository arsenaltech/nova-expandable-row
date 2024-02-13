<template>
  <span ref="indexField" class="text-center">
    <Teleport :to="toggleTeleportTarget" v-if="isMounted" :disabled="!this.field.moveToActions">
      <div
        class="relative rounded active:outline-none active:ring font-bold focus:outline-none focus:ring focus:ring-primary-200 dark:focus:ring-gray-600 -order-1 flex items-center justify-center"
        :class="this.field.moveToActions ? 'pl-7' : 'px-7'" style="order: -1" @click.stop="toggleExpandedRow">

        <span v-if="this.field.showIcon" class="absolute left-0  mr-2 p-2">
          <component :is="`heroicons-solid-${rowExpanded ? this.field.iconForHideExpand : this.field.iconForExpand}`"
            height="21" width="21" />
        </span>


        <span v-if="!this.field.moveToActions">{{ this.field.toggleLabel }}</span>

        <button class="toolbar-button flex items-center cursor-pointer select-none">
          <div class="py-0.5 px-2 rounded">

          </div>
        </button>
      </div>
    </Teleport>
  </span>

  <Teleport :to="to" v-if="isMounted" class="bg-gray-50 border-b border-gray-200 dark:border-gray-700">
    <td :colspan="2" v-if="rowExpanded && resource" class="td-fit"></td>
    <td :colspan="columnCount - 2" v-if="rowExpanded && resource" class="td-fit">
      <div style="padding-bottom: 0;">
        <div class="px-2 py-4 space-y-4">
          <div :key="index" v-for="(field, index) in resource.fields" :index="index">
            <h4 class="font-bold"><span>{{ field.name }}</span></h4>
            <button v-for="value in field.value" type="button" class="w-full">
              <div class="font-semibold flex flex-row justify-start items-center" v-html="value.display"></div>
            </button>

            <button v-if="!field.value.length" type="button" class="w-full">
              <div class="text-xs font-semibold">-</div>
            </button>
          </div>

        </div>
      </div>
    </td>
  </Teleport>
</template>

<script>

export default {
  props: ['resourceName',
    'field'],

  data: () => ({
    loading: true,
    title: null,
    resource: null,


    isMounted: false,
    columnCount: 1,
    rowExpanded: false,

    trNext: null,
  }),


  mounted() {
    this.$nextTick(() => {

      // Create a table row to teleport the expanded row to
      this.createTableRow();
    });
  },
  methods: {
    createTableRow() {
      const indexField = this.$refs.indexField;
      const tr = indexField.closest('tr');
      const td = indexField.closest('td');


      // TODO: remove the column if moving trigger to actions
      // const siblings = Array.from(tr.children);    
      // const index = siblings.indexOf(td);
      // td.remove();
      // const th = tr.closest('table').querySelector('thead > tr');
      // console.log('th', th);

      this.columnCount = tr.querySelectorAll('td').length;
      this.trNext = document.createElement('tr');
      this.trNext.classList.add('bg-gray-100');
      this.trNext.classList.add('dark:bg-gray-800');

      tr.insertAdjacentElement('afterend', this.trNext);
      this.to = this.trNext;


      const actionItmes = tr.querySelector('.flex.items-center.justify-end.space-x-0.text-gray-400');
      this.toggleTeleportTarget = actionItmes;


      this.isMounted = true;

    },

    toggleExpandedRow() {
      this.rowExpanded = !this.rowExpanded;
      this.trNext.classList.toggle('rowExpanded');

      if (this.field.expandingData) {
        this.resource = {
          fields: [],
        };
        this.resource.fields = this.field.expandingData;
        this.loading = false;
      } else if (!this.resource) {
        this.getResource();
      }
    },

    getResource() {
      Nova.request().get(
        `/nova-api/${this.resourceName}/${this.$parent.resource.id.value}/preview`
      )
        .then(({ data: { title, resource } }) => {
          this.title = title
          this.resource = resource
          this.loading = false
        })
        .catch(error => {
          console.log(error);

          if (error.response.status >= 500) {
            Nova.$emit('error', error.response.data.message)
            return
          }

          if (error.response.status === 404) {
            Nova.visit('/404')
            return
          }

          if (error.response.status === 403) {
            Nova.visit('/403')
            return
          }

          if (error.response.status === 401) return Nova.redirectToLogin()

          Nova.error(this.__('This resource no longer exists'))

          Nova.visit(`/resources/${this.resourceName}`)
        })
    }

  },
}
</script>
<style>
.bg-gray-100.rowExpanded {
  background: #fff;
}

:is(.dark .dark\:bg-gray-800).rowExpanded {
  background: #1e293b;
}

.rotate-180 {
  transform: rotate(180deg);
}

.px-7 {
  padding-left: 1.75rem;
  padding-right: 1.75rem;
}

.pl-7 {
  padding-left: 1.75rem;
}

/* Add padding bottom only to the last expanded row */
.rowExpanded>td:last-child>div {
  padding-bottom: 1rem;
}

.bg-gray-100.dark\:bg-gray-800.rowExpanded:has(~ .rowExpanded)>td:last-child>div {
  padding-bottom: 0rem !important;
}
</style>
