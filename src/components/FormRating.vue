<script setup lang="ts">
import { ref } from "vue";
import { useRatingStore } from '@/stores/rating';
import { useUserStore } from "@/stores/user";
import RatingView from '../components/RatingView.vue';
import BaseLink from '../components/BaseLink.vue';
import {
    ElButton,
} from 'element-plus';

const ratingStore = useRatingStore();
const userStore = useUserStore();

userStore.updateRatingCount();

</script>

<template>
    <h1 style="margin: 0">Image Rating</h1>
    <div>Rate images based on aesthetics to gain kudos and help <BaseLink href="https://laion.ai/">LAION</BaseLink> - the non-profit who helped train Stable Diffusion - improve their datasets!</div>
    <div v-if="userStore.apiKey === '0000000000' || userStore.apiKey === ''">You have rated a total of <strong>{{ ratingStore.imagesRated }}</strong> images! <BaseLink router href="/options">Sign in</BaseLink> using your API key to start earning kudos.</div>
    <div v-else>From rating a total of <strong>{{ userStore.ratingCount }}</strong> images, you have gained <strong>{{ userStore.ratingKudos }}</strong> kudos!</div>
    <el-button
        @click="() => ratingStore.updateRatingInfo()"
        v-if="!ratingStore.currentRatingInfo.id"
        :disabled="ratingStore.submitted"
        style="margin-top: 10px"
        size="large"
    >{{ ratingStore.submitted ? "Loading image..." : "Start rating!"}}</el-button>
    <RatingView
        :id="ratingStore.currentRatingInfo.id || ''"
        :image-source="ratingStore.currentRatingInfo.url || ''"
        :submitted="ratingStore.submitted"
        @onRatingSubmit="ratingStore.submitRating"
    />
</template>
