<template>
  <component :is="getComponent(field)" v-bind="$props" read-only></component>
</template>

<script>
import rowEditField from '@baserow/modules/database/mixins/rowEditField'
import RowEditFieldDateReadOnly from '@baserow/modules/database/components/row/RowEditFieldDateReadOnly'
import RowEditFieldText from '@baserow/modules/database/components/row/RowEditFieldText'
import RowEditFieldBoolean from '@baserow/modules/database/components/row/RowEditFieldBoolean'
import RowEditFieldNumber from '@baserow/modules/database/components/row/RowEditFieldNumber'

export default {
  name: 'RowEditFieldFormula',
  components: {
    RowEditFieldDateReadOnly,
    RowEditFieldText,
    RowEditFieldBoolean,
    RowEditFieldNumber,
  },
  mixins: [rowEditField],
  methods: {
    getComponent(field) {
      return {
        date: RowEditFieldDateReadOnly,
        text: RowEditFieldText,
        boolean: RowEditFieldBoolean,
        number: RowEditFieldNumber,
        invalid: RowEditFieldText,
        char: RowEditFieldText,
        date_interval: RowEditFieldText,
      }[field.formula_type]
    },
  },
}
</script>
