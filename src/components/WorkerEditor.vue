<script setup lang=ts>
import { ref } from "vue";
import {
    ElButton,
    ElDialog,
    ElForm,
    ElSwitch,
    ElFormItem,
    ElInput,
    ElMessageBox
} from 'element-plus';
import WorkerBox from './WorkerBox.vue';
import FormSelect from './FormSelect.vue';
import type { WorkerDetailsStable } from "@/types/stable_horde";
import { useWorkerStore } from "@/stores/workers";
import { useUserStore } from "@/stores/user";
import { validateResponse } from "@/utils/validate";
import { BASE_URL_STABLE } from "@/constants";

// eslint-disable-next-line @typescript-eslint/no-unused-vars
const props = defineProps<{
    worker: WorkerDetailsStable;
}>();

const workerStore = useWorkerStore();
const userStore = useUserStore();

async function updateWorkerOptions() {
    const response = await fetch(`${BASE_URL_STABLE}/api/v2/workers/${props.worker?.id}`, {
        method: "PUT",
        body: JSON.stringify(workerOptionsChange.value),
        headers: {
            "Content-Type": "application/json",
            apikey: userStore.apiKey
        }
    });
    const resJSON = await response.json();
    if (response.status === 403) {
        workerStore.updateWorkers()
        return resJSON;
    }
    if (response.status === 500) {
        // Hack cause Horder is always reporting 500,
        // if paused argument is added that is not the case but
        // worker name is not changeable anymore due to
        // paused being only modifyable as an moderator
        return;
    }
    if (!validateResponse(response, resJSON, 200, "Failed to modify worker")) return false;
    workerStore.updateWorkers()
    return resJSON;
}

let deleteTimer = ref<any>(undefined);

function deleteWorker() {
    ElMessageBox.confirm(
        "This action will permanently delete this worker. Continue?",
        'Delete Worker?',
        {
            confirmButtonText: 'Delete',
            cancelButtonText: 'Cancel',
            type: 'warning',
        }
    ).then(() => {
        deleteTimer.value = setTimeout(async () => {
            const response = await fetch(`${BASE_URL_STABLE}/api/v2/workers/${props.worker?.id}`, {
                method: "DELETE",
                headers: {
                    apikey: userStore.apiKey
                }
            });
            const resJSON = await response.json();
            if (!validateResponse(response, resJSON, 200, "Failed to delete worker")) return false;
            workerStore.updateWorkers();
            dialogOpen.value = false;
        }, 60 * 1000)
    })
}

function cancelDeleteWorker() {
    clearTimeout(deleteTimer.value);
    deleteTimer.value = undefined;
}

const dialogOpen = ref(false);
const workerOptionsChange = ref({
    maintenance: props.worker?.maintenance_mode,
    maintenance_msg: "",
    info: props.worker.info,
    name: props.worker.name,
    team: props.worker.team?.id === null ? '' : props.worker.team?.id
})
</script>

<template>
    <WorkerBox
        v-if="worker != undefined"
        :worker="worker"
    >
        <template #header>
            <el-button @click="dialogOpen = true">Edit Worker</el-button>
            <el-dialog
                v-model="dialogOpen"
                :title="worker.name"
                style="height: 500px; width: 600px;"
                align-center
            >
                <el-form label-width="140px" :model="workerOptionsChange" label-position="left" @submit.prevent>
                    <el-form-item label="Change Name">
                        <div style="font-size: 13px; word-break: keep-all;">Make sure to stop the worker first and then edit bridgeData.py!</div>
                        <el-input
                            v-model="workerOptionsChange.name"
                            placeholder="Enter new name here" 
                            style="width: 80%; min-width: 200px"
                        />
                        <el-button @click="updateWorkerOptions">Submit</el-button>
                    </el-form-item>
                    <el-form-item label="Info">
                        <el-input
                            v-model="workerOptionsChange.info"
                            :autosize="{ minRows: 2, maxRows: 10 }"
                            clearable
                            resize="none"
                            type="textarea"
                            style="width: 80%; word-break:keep-all; min-width: 200px;"
                            maxlength="1000"
                            placeholder="Enter new info here"
                        />
                        <el-button @click="updateWorkerOptions">Submit</el-button>
                    </el-form-item>
                    <FormSelect
                        label="Team"
                        prop="team"
                        v-model="workerOptionsChange.team"
                        :options="[
                            {label: 'None', value: ''},
                            ...workerStore.teams.map(el => {return {label: el.name, value: el.id}})
                        ]"
                        :change="updateWorkerOptions"
                    />
                    <el-form-item label="Maintenance Mode">
                        <el-switch v-model="workerOptionsChange.maintenance" @change="updateWorkerOptions" />
                    </el-form-item>
                    <el-form-item label="Delete Worker">
                        <el-button type="danger" v-if="deleteTimer == undefined" @click="deleteWorker">Remove</el-button>
                        <el-button type="danger" v-if="deleteTimer != undefined" @click="cancelDeleteWorker">Cancel Remove (60s timer)</el-button>
                    </el-form-item>
                </el-form>
            </el-dialog>
        </template>
    </WorkerBox>
</template>