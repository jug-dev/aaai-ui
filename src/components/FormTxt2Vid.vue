<script setup lang="ts">

import { useVideoGeneratorStore } from '@/stores/VideoGenerator';
import { ElForm, ElButton, ElCard, ElProgress, ElRow, ElCol } from 'element-plus';
import FormSlider from './FormSlider.vue';
import FormPun from './FormPuns.vue';
import FormSelect from './FormSelect.vue';
import FormSeedVideo from './FormSeedVideo.vue';
import FormPromptInputVideo from './FormPromptInputVideo.vue';
import BaseLink from '@/components/BaseLink.vue';

const store = useVideoGeneratorStore();

function getAspectRatio(isWidth: boolean) {
    let width = (store.params.width || 1);
    let height = (store.params.height || 1);
    let dividend = 0;
    let divisor = 0;
    if(width == height) {
        return "(1:1)";
    } else {
        if(height>width) {
            dividend = height;
            divisor = width;
        }
        if(width>height) {
            dividend = width;
            divisor = height;
        }
        var gcd = -1;
        let remainder = 0;
        while(gcd == -1){
            remainder = dividend % divisor;
            if(remainder == 0) {
                gcd = divisor;
            } else {
                dividend = divisor;
                divisor = remainder;
            }
        }
        var hr = width / gcd;
        var vr = height / gcd;
        if(isWidth)
            return "(" + hr.toFixed(0) + ":" + vr.toFixed(0) + ")";
        else
            return "(" + vr.toFixed(0) + ":" + hr.toFixed(0) + ")";
    }
}

</script>
<template>
    <el-form
            label-position="left"
            label-width="140px"
            :model="store"
            class="container"
            :rules="store.generateFormBaseRules"
            @submit.prevent
        >
            <div class="sidebar">
                <form-prompt-input-video />  
                <FormSeed-video v-if="store.params.model != 'Kandinsky'" /> 
                <form-select v-if="store.params.model != 'Kandinsky'" label="Sampler" prop="sampler" v-model="store.params.sampler" :options="store.AvailableSamplers" />
                <form-slider label="Steps" prop="steps" v-model="store.params.steps" :min="store.minSteps" :max="store.maxSteps" info="Keep step count between 30 to 50 for optimal generation times. Coherence typically peaks between 60 and 90 steps, with a trade-off in speed." /> 
                <form-slider v-if="store.params.model == 'Kandinsky'" label="Prior Steps" prop="priorSteps" v-model="store.params.prior_steps" :min="store.minSteps" :max="store.maxSteps" />
                <form-slider :label="`Width ` + getAspectRatio(true)" prop="width" v-model="store.params.width" :min="store.minWidth" :max="store.maxWidth" :step="64" />
                <form-slider label="Height" prop="height" v-model="store.params.height" :min="store.minHeight" :max="store.maxHeight" :step="64" />
                <form-slider label="Guidance" prop="cfgScale" v-model="store.params.cfg_scale" :min="store.minCfgScale" :max="store.maxCfgScale" :step="0.5" info="Higher values will make the AI respect your prompt more. Lower values allow the AI to be more creative." />
                <form-slider v-if="store.params.model == 'Kandinsky'" label="Prior Guidance" prop="priorCfgScale" v-model="store.params.prior_cfg_scale" :min="store.minCfgScale" :max="store.maxCfgScale" :step="0.5" />
                <form-slider v-if="store.params.model != 'Kandinsky'" label="Desired FPS" prop="desiredFPS" v-model="store.params.fps" :min="store.minFPS" :max="store.maxFPS" />
                <form-slider v-if="store.params.model != 'Kandinsky'" label="Desired Length" prop="desiredLength" v-model="store.params.desired_duration" :min="store.minDuration" :max="store.maxDuration" />
                <form-select label="Model" prop="model" v-model="store.params.model" :options="store.AvailableModels" :filterable="true" />
                <form-select v-if="store.params.model != 'Kandinsky'" label="Interpolate" prop="interpolate" v-model="store.params.interpolate" :options="store.AvailableInterpolations" />
                <form-slider label="Times to Interpolate" v-if="store.params.interpolate !== 'None' && store.params.model != 'Kandinsky'" prop="timesToInterpolate" v-model="store.params.timestointerpolate" :min="store.minTimestointerpolate" :max="store.maxTimestointerpolate" />
            </div>
            <div class="main">
                <el-button
                    type="primary"
                    :disabled="store.generating || store.generateLock || store.params.model == 'Kandinsky'"
                    style="width: 30%;font-size: 0.9em;"
                    @click="store.generateClicked()"
                >
                    Generate Video ({{ store.getTime() }})
                </el-button>
                <el-button
                    type="primary"
                    :disabled="store.generating || store.generateLock"
                    style="width: 30%;font-size: 0.9em;"
                    @click="store.generateImageClicked()"
                >
                    Generate Frame
                </el-button>
                <el-button
                    :type="store.generating ? 'danger' : 'info'"
                    :plain="!store.generating"
                    style="width: 20%"
                    :disabled="store.cancelled || !store.generating"
                    @click="store.cancelled = true"
                > Cancel
                </el-button>
                <el-button
                    :type="(!store.generating && store.videoUrl !== 'none') ? 'danger' : 'info'"
                    :plain="store.generating || store.videoUrl === 'none'"
                    :disabled="store.generating || store.videoUrl === 'none'"
                    style="width: 20%"
                    @click="store.deleteVideo()"
                > 
                    Delete
                </el-button>
            </div>
            <div class="image center-horizontal">
                <el-card class="center-both generated-video" >
                    <div class="noticeBox" v-if="!store.generating && store.videoUrl === 'none'">
                        <div class="genNotice" v-if="store.generateLock">
                            <strong>ATTENTION</strong>
                            <hr/>
                            Can not accept more generation requests!<br/>
                            {{ store.generateMsg }}<br/>
                            To find out more join our <BaseLink href="https://discord.gg/ugEqPP5wMT">Discord</BaseLink>!
                        </div>
                        <div>
                            <FormPun />    
                        </div>
                    </div>
                    <div v-if="!store.generating && store.videoUrl !== 'none'" class="player">
                        <video controls v-if="store.videoUrl.endsWith('.mp4')">
                            <source :src=store.videoUrl type="video/mp4" />
                        </video>
                        <img v-if="store.videoUrl.endsWith('.png')" :src=store.videoUrl type="image/png" style="max-width: 100%;max-height: 100%;" />
                    </div>                           
                    <div v-if="store.generating" style="text-align: center;">
                        <el-progress
                            type="circle"
                            :percentage="store.progress"
                            :width="200"
                        >
                            <template #default>
                                <span>{{ store.getProgressWriting() }}</span><br>
                            </template>
                        </el-progress>
                        <div style="font-size: 15px; padding: 8px; margin-top: 10px; background-color: var(--el-color-info-light-9); border-radius: 5px">
                            <div style="font-size: 18px">Generation Status</div>
                            <div>{{ store.queueStatus }}</div>
                        </div>
                    </div>
                </el-card> 
            </div>
    </el-form>
</template>

<style scoped>

.generated-video {
    width: 100%;
}

.player video {
    max-width: 100%;
    max-height: 100%;
}

.noticeBox {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}

.genNotice {
    color: black;
    padding: 20px;
    border-radius: 5px;
	background: #FFFB00; 
	border: solid #000000 3px; 
	box-shadow: 2px 2px 22px rgba(0, 0, 0, 0.5) inset; 
	-webkit-box-shadow: 2px 2px 22px rgba(0, 0, 0, 0.5) inset; 
	-moz-box-shadow: 2px 2px 22px rgba(0, 0, 0, 0.5) inset; 
}

.genNotice hr {
    border-color: black;
}

</style>