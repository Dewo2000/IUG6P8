<template>
    <h4>Usuario <span class="name">{{ user.firstName }} {{ user.lastName }}</span></h4>

    <table>
        <tbody>
            <tr>
                <th>Rol</th>
                <td>{{ user.userRole }} </td>
            </tr>
            <tr>
                <th>Créditos</th>
                <td>imparte {{ user.assignedCredits }} de {{ user.maxCredits }}</td>
            </tr>
            <tr>
                <th>Grupos a los que da clase</th>
                <td v-if="user.groups.length">
                    <span
                    v-for="group in user.groups"
                    :key="group.id"
                    class="badge rounded-pill text-bg-secondary me-1"
                    :title="'Asignatura: ' + gState.resolve(gState.resolve(group).subjectId).name + ' Grado: ' + gState.resolve(gState.resolve(group).subjectId).degree ">
                        {{ formatNiceGroup(group) }}
                    </span>
                </td>
                <td v-else> (ninguno) </td>
            </tr>
        </tbody>
    </table>

    <SortableGrid :data="addSlotCols(slots)" :columns="slotColumns" 
      :filter="{ all: '', fields: [] }" v-model:sorter="sorter" />
      <div class="form-check">
      <input 
        type="checkbox" 
        id="firstSemester" 
        v-model="showFirstSemester" 
        @change="toggleSemester" 
        class="form-check-input" 
      />
      <label for="firstSemester" class="form-check-label">Primer Cuatrimestre</label>
    </div>
    <div class="form-check">
      <input 
        type="checkbox" 
        id="secondSemester" 
        v-model="showSecondSemester" 
        @change="toggleSemester" 
        class="form-check-input" 
      />
      <label for="secondSemester" class="form-check-label">Segundo Cuatrimestre</label>
    </div>

    <!-- Tablas dinámicas -->
    <div v-if="showFirstSemester">
  <h3>Horario - Primer Cuatrimestre</h3>
  <TimeTable :key="'first-' + forceUpdateKey" :slots="firstSemesterSlots" />
</div>
<br>
<div v-if="showSecondSemester">
  <h3>Horario - Segundo Cuatrimestre</h3>
  <TimeTable 
    :key="'second-' + forceUpdateKey" 
    :slots="secondSemesterSlots" 
  />
</div>

    <h5>Acciones</h5>
    <div class="btn-group">
        <button @click="$emit('editUser')" class="btn btn-outline-success" title="Editar usuario">✏️</button>
        <button @click="$emit('rmUser')" class="btn btn-outline-danger" title="Eliminar usuario">🗑️</button>
    </div>
</template>

<script setup>
import SortableGrid from './SortableGrid.vue';
import TimeTable from './TimeTable.vue';

import { gState, semesterNames, weekDayNames } from '../state.js';
import { ref, computed } from 'vue'

defineEmits([
  'filterUser',
  'editUser',
  'rmUser',
])

const props = defineProps({
    user: Object // see definition of User in ../model.js
})

let sorter = ref([{key: "weekDay", order: 1}])


const showFirstSemester = ref(false);
const showSecondSemester = ref(false);

const forceUpdateKey = ref(0);

const toggleSemester = () => {
  forceUpdateKey.value++;
};

const firstSemesterSlots = computed(() =>
  showFirstSemester.value ? slots.value.filter(slot => slot.semester === 'FALL') : []
);

const secondSemesterSlots = computed(() =>
  showSecondSemester.value ? slots.value.filter(slot => slot.semester === 'SPRING') : []
);


const slots = computed(() => {
    return slotsOfAllGroups(props.user.groups);
});


const slotsOfAllGroups = (groups) => {
    return groups.flatMap(group => {
        const resolvedGroup = gState.resolve(group);
        return resolvedGroup.slots.map(slot => gState.resolve(slot));
    });
};

const slotColumns = [
    { key: 'niceGroup', display: 'Grupo', type: 'String' },
    {
        key: 'weekDay', display: 'Día', type: 'Enum',
        values: gState.model.WeekDay,
        displayValues: weekDayNames
    },
    {
        key: 'semester', display: 'Cuatr.', type: 'Enum',
        values: gState.model.Semester,
        displayValues: semesterNames
    },
    { key: 'niceStart', display: 'Inicio', type: 'Time' },
    { key: 'niceEnd', display: 'Final', type: 'Time' },
    { key: 'location', display: 'Lugar', type: 'String' },
]

const addSlotCols = (ss) => {
    for (let s of ss) {
        s.niceGroup = formatNiceGroup(s.groupId)
        s.niceStart = formatTime(s.startTime)
        s.niceEnd = formatTime(s.endTime)
    }
    return ss;
}

// 1450 => "14:50"
const formatTime = t => `${Math.floor(t / 100)}:` + `0${t % 100}`.slice(-2)

const formatNiceGroup = groupId => {
    const group = gState.resolve(groupId);
    const subject = gState.resolve(group.subjectId);
    return `${subject.short}:${group.name}`
}

</script>