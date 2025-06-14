<script lang="ts">
    import type {PageProps} from './$types';
    import {Modal, Heading, Rating, Card, Button, Textarea, P} from 'flowbite-svelte';
    import {addComment, addProgressPresentation} from '$lib/strapiRepository';
    import {notifyUserDataChanged, userStore} from '$lib/store';
    import type {AuthorComment} from '$lib';
    import {getPresentationSummary} from "$lib/deepseekRepository";
    import {onMount} from "svelte";

    const {data}: PageProps = $props();
    onMount(() => {
        if ($userStore === null) return;
        console.log(`User visits lecture ${data.presentation.presentationName} with courseId = ${data.presentation.courseDocumentId}`);
        addProgressPresentation($userStore, data.presentation.courseDocumentId, data.presentation.id)
            .then(() => notifyUserDataChanged($userStore));
    })

    let commentText: string = $state('');
    let comments: AuthorComment[] = data.presentation.comments;

    function averageRating() {
        const length = data.presentation.votedAuthors.length;
        if (length === 0) return 0;
        return (
            data.presentation.votedAuthors
                .map((person) => person.authorScore)
                .reduce((sum, value) => sum + value, 0) / length
        );
    }

    const dynamicManifest = {
        name: `${data.presentation.presentationName} | ITMO Frontend Courses`,
        short_name: "ITMO Lecture",
        description: data.presentation.presentationDescription,
        start_url: `/lectures/${data.presentation.id}`,
        theme_color: "#FE8A70",
        background_color: "#ffffff"
    };

    let scrollingModal = $state(false);
    let summaryPromise = $state<Promise<string> | null>(null);

    async function generateSummary() {
        summaryPromise = getPresentationSummary(data.presentation.presentationUrl);
    }

    async function handleComment() {
        if (!$userStore?.name) {
            return;
        }

        await addComment(commentText, $userStore.id, data.presentation.id);
        commentText = '';

        window.location.reload();
    }

    function getPresentationUrl(rawUrl: string): string {
        if (rawUrl.includes('vercel')) {
            return rawUrl;
        }
        return `https://docs.google.com/presentation/d/${rawUrl}/embed?start=false&loop=false`;
    }
</script>

<svelte:head>
    <title>{data.presentation.presentationName} | Lectures</title>
    <link rel="manifest" href="manifest.json" data-sveltekit-preload-data="true"/>
    <meta name="theme-color" content="#FE8A70"/>
</svelte:head>

<div class="w-[80%] mx-auto px-4 py-8">
    <div class="flex flex-col md:flex-row justify-between items-start mb-8 gap-4">
        <div class="flex-1">
            <h1 class="lecture-name">
                {data.presentation.presentationName}
            </h1>
            <p class="text-lg text-[#fcefe8] dark:text-gray-300">
                {data.presentation.presentationDescription}
            </p>
        </div>
        <div class="w-full md:w-auto md:text-right">
            <p class="text-sm text-[#fcefe8] dark:text-gray-400">Average rating</p>
            <Rating id="lecture-rating" total={5} rating={averageRating()} class="justify-start md:justify-end"/>
            <p class="text-xs text-[#fcefe8] dark:text-gray-400 mt-1">
                Based on {data.presentation.votedAuthors.length} votes
            </p>
        </div>
    </div>

    <div class="w-full mb-8">
        <div class="w-full aspect-video bg-gray-100 dark:bg-gray-800 rounded-lg overflow-hidden">
            <iframe
                    title="Lecture presentation"
                    class="w-full h-full"
                    src='{getPresentationUrl(data.presentation.presentationUrl)}'
            >
            </iframe>
        </div>

        <div class="mt-4">
            <a
                    class="inline-flex items-center px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
                    download="presentation.pdf"
                    href={`https://docs.google.com/presentation/d/${data.presentation.presentationUrl}/export/pdf`}
            >
                <svg
                        class="w-5 h-5 mr-2"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                        xmlns="http://www.w3.org/2000/svg"
                >
                    <path
                            d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                    ></path>
                </svg>
                Скачать PDF
            </a>

            <a
                    class="inline-flex items-center px-4 py-2 ml-2 bg-green-500 text-white rounded hover:bg-green-600 transition-colors"
                    download="presentation.pptx"
                    href={`https://docs.google.com/presentation/d/${data.presentation.presentationUrl}/export/pptx`}
            >
                <svg
                        class="w-5 h-5 mr-2"
                        fill="none"
                        stroke="currentColor"
                        viewBox="0 0 24 24"
                        xmlns="http://www.w3.org/2000/svg"
                >
                    <path
                            d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                    ></path>
                </svg>
                Скачать PPTX
            </a>

            {#if summaryPromise}
                {#await summaryPromise}
                    <a
                            class="inline-flex items-center px-4 py-2 ml-2 bg-blue-300 text-white rounded transition-colors"
                    >
                        <svg
                                class="w-5 h-5 mr-2"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 57 42"
                                xmlns="http://www.w3.org/2000/svg"
                        >
                            <path
                                    fill="#4D6BFE"
                                    d="M55.6128 3.47122C55.0175 3.17947 54.7611 3.73538 54.413 4.01785C54.2939 4.10892 54.1932 4.22732 54.0924 4.3367C53.2223 5.2659 52.2057 5.87649 50.8776 5.8035C48.9359 5.69412 47.2781 6.30472 45.8126 7.78982C45.5012 5.95853 44.4663 4.86502 42.8909 4.1636C42.0667 3.7991 41.2332 3.4346 40.6561 2.64188C40.2532 2.07718 40.1432 1.44852 39.9418 0.828889C39.8135 0.455354 39.6853 0.0725423 39.2548 0.00882154C38.7877 -0.0639323 38.6045 0.32767 38.4213 0.655795C37.6886 1.99515 37.4047 3.47122 37.4321 4.96536C37.4962 8.32742 38.9159 11.0059 41.7369 12.9102C42.0575 13.1289 42.1399 13.3474 42.0392 13.6665C41.8468 14.3225 41.6178 14.9602 41.4164 15.6162C41.2881 16.0354 41.0957 16.1265 40.647 15.9441C39.0991 15.2974 37.7618 14.3406 36.5803 13.1836C34.5745 11.2429 32.761 9.10184 30.4988 7.42532C29.9675 7.03348 29.4363 6.66922 28.8867 6.32278C26.5786 4.08157 29.189 2.241 29.7935 2.02249C30.4254 1.79471 30.0133 1.01102 27.9708 1.02029C25.9283 1.02933 24.0599 1.71268 21.6786 2.62381C21.3306 2.76053 20.9641 2.86063 20.5886 2.94266C18.4271 2.53274 16.1831 2.44144 13.8384 2.70584C9.42368 3.19778 5.89755 5.28421 3.30551 8.84671C0.19138 13.1289 -0.541286 17.9941 0.356297 23.0691C1.29965 28.4172 4.02902 32.8452 8.22385 36.3076C12.5745 39.8972 17.5845 41.6558 23.2997 41.3186C26.771 41.1182 30.6361 40.6536 34.9958 36.9636C36.0948 37.5103 37.2489 37.7288 39.1632 37.8928C40.6378 38.0295 42.0575 37.8201 43.1565 37.5923C44.8784 37.2278 44.7594 35.6333 44.1366 35.3418C39.09 32.9912 40.1981 33.9478 39.1907 33.1733C41.7552 30.1394 45.6204 26.9868 47.1316 16.7732C47.2506 15.9624 47.1499 15.4521 47.1316 14.7961C47.1224 14.3953 47.214 14.2405 47.672 14.1948C48.9359 14.0491 50.1632 13.7029 51.2898 13.0833C54.5596 11.2976 55.8784 8.3638 56.1898 4.84695C56.2357 4.30936 56.1807 3.75345 55.6128 3.47122ZM27.119 35.123C22.2281 31.2783 19.856 30.0117 18.8759 30.0664C17.96 30.1211 18.1249 31.1689 18.3263 31.8523C18.537 32.5264 18.8118 32.9912 19.1964 33.5833C19.462 33.9751 19.6453 34.5581 18.9309 34.9956C17.3555 35.9705 14.6169 34.6675 14.4886 34.6038C11.3014 32.7268 8.63608 30.2485 6.75839 26.8594C4.94492 23.5974 3.89169 20.0989 3.71762 16.3633C3.67185 15.4614 3.93747 15.1423 4.83505 14.9785C6.01657 14.7598 7.23471 14.7141 8.41623 14.8872C13.408 15.6162 17.6577 17.8484 21.2206 21.3835C23.2539 23.397 24.7926 25.8025 26.3772 28.1531C28.0624 30.6494 29.8759 33.0276 32.184 34.9773C32.9991 35.6606 33.6494 36.1799 34.2722 36.5627C32.3947 36.7722 29.2622 36.8179 27.119 35.123ZM29.4637 20.0442C29.4637 19.6433 29.7843 19.3245 30.1874 19.3245C30.2789 19.3245 30.3613 19.3425 30.4346 19.3699C30.5354 19.4065 30.627 19.4612 30.7002 19.543C30.8285 19.6707 30.9017 19.8528 30.9017 20.0442C30.9017 20.4451 30.5812 20.7639 30.1782 20.7639C29.7751 20.7639 29.4637 20.4451 29.4637 20.0442ZM36.7452 23.7798C36.2781 23.9712 35.811 24.135 35.3622 24.1533C34.6661 24.1897 33.9059 23.9072 33.4938 23.561C32.8527 23.0234 32.3947 22.7229 32.2023 21.7844C32.1199 21.3835 32.1656 20.7639 32.239 20.4087C32.4038 19.6433 32.2206 19.1514 31.6803 18.7048C31.2406 18.3403 30.6819 18.2402 30.0682 18.2402C29.8392 18.2402 29.6287 18.1399 29.4729 18.0579C29.2164 17.9304 29.0059 17.6116 29.2073 17.2197C29.2714 17.0923 29.5829 16.7825 29.6561 16.7278C30.4896 16.2539 31.4513 16.4089 32.3397 16.7642C33.1641 17.1013 33.7869 17.7209 34.6844 18.5955C35.6003 19.6523 35.7651 19.9441 36.2872 20.7366C36.6995 21.3562 37.075 21.9939 37.3314 22.7229C37.4871 23.1785 37.2856 23.552 36.7452 23.7798Z"
                            ></path>
                        </svg>
                        Генерация конспекта...
                    </a>
                {:then summary}
                    <button
                            class="inline-flex items-center px-4 py-2 ml-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
                            onclick={() => (scrollingModal = true)}
                    >
                        <svg
                                class="w-5 h-5 mr-2"
                                fill="none"
                                stroke="currentColor"
                                viewBox="0 0 57 42"
                                xmlns="http://www.w3.org/2000/svg"
                        >
                            <path
                                    fill="#4D6BFE"
                                    d="M55.6128 3.47122C55.0175 3.17947 54.7611 3.73538 54.413 4.01785C54.2939 4.10892 54.1932 4.22732 54.0924 4.3367C53.2223 5.2659 52.2057 5.87649 50.8776 5.8035C48.9359 5.69412 47.2781 6.30472 45.8126 7.78982C45.5012 5.95853 44.4663 4.86502 42.8909 4.1636C42.0667 3.7991 41.2332 3.4346 40.6561 2.64188C40.2532 2.07718 40.1432 1.44852 39.9418 0.828889C39.8135 0.455354 39.6853 0.0725423 39.2548 0.00882154C38.7877 -0.0639323 38.6045 0.32767 38.4213 0.655795C37.6886 1.99515 37.4047 3.47122 37.4321 4.96536C37.4962 8.32742 38.9159 11.0059 41.7369 12.9102C42.0575 13.1289 42.1399 13.3474 42.0392 13.6665C41.8468 14.3225 41.6178 14.9602 41.4164 15.6162C41.2881 16.0354 41.0957 16.1265 40.647 15.9441C39.0991 15.2974 37.7618 14.3406 36.5803 13.1836C34.5745 11.2429 32.761 9.10184 30.4988 7.42532C29.9675 7.03348 29.4363 6.66922 28.8867 6.32278C26.5786 4.08157 29.189 2.241 29.7935 2.02249C30.4254 1.79471 30.0133 1.01102 27.9708 1.02029C25.9283 1.02933 24.0599 1.71268 21.6786 2.62381C21.3306 2.76053 20.9641 2.86063 20.5886 2.94266C18.4271 2.53274 16.1831 2.44144 13.8384 2.70584C9.42368 3.19778 5.89755 5.28421 3.30551 8.84671C0.19138 13.1289 -0.541286 17.9941 0.356297 23.0691C1.29965 28.4172 4.02902 32.8452 8.22385 36.3076C12.5745 39.8972 17.5845 41.6558 23.2997 41.3186C26.771 41.1182 30.6361 40.6536 34.9958 36.9636C36.0948 37.5103 37.2489 37.7288 39.1632 37.8928C40.6378 38.0295 42.0575 37.8201 43.1565 37.5923C44.8784 37.2278 44.7594 35.6333 44.1366 35.3418C39.09 32.9912 40.1981 33.9478 39.1907 33.1733C41.7552 30.1394 45.6204 26.9868 47.1316 16.7732C47.2506 15.9624 47.1499 15.4521 47.1316 14.7961C47.1224 14.3953 47.214 14.2405 47.672 14.1948C48.9359 14.0491 50.1632 13.7029 51.2898 13.0833C54.5596 11.2976 55.8784 8.3638 56.1898 4.84695C56.2357 4.30936 56.1807 3.75345 55.6128 3.47122ZM27.119 35.123C22.2281 31.2783 19.856 30.0117 18.8759 30.0664C17.96 30.1211 18.1249 31.1689 18.3263 31.8523C18.537 32.5264 18.8118 32.9912 19.1964 33.5833C19.462 33.9751 19.6453 34.5581 18.9309 34.9956C17.3555 35.9705 14.6169 34.6675 14.4886 34.6038C11.3014 32.7268 8.63608 30.2485 6.75839 26.8594C4.94492 23.5974 3.89169 20.0989 3.71762 16.3633C3.67185 15.4614 3.93747 15.1423 4.83505 14.9785C6.01657 14.7598 7.23471 14.7141 8.41623 14.8872C13.408 15.6162 17.6577 17.8484 21.2206 21.3835C23.2539 23.397 24.7926 25.8025 26.3772 28.1531C28.0624 30.6494 29.8759 33.0276 32.184 34.9773C32.9991 35.6606 33.6494 36.1799 34.2722 36.5627C32.3947 36.7722 29.2622 36.8179 27.119 35.123ZM29.4637 20.0442C29.4637 19.6433 29.7843 19.3245 30.1874 19.3245C30.2789 19.3245 30.3613 19.3425 30.4346 19.3699C30.5354 19.4065 30.627 19.4612 30.7002 19.543C30.8285 19.6707 30.9017 19.8528 30.9017 20.0442C30.9017 20.4451 30.5812 20.7639 30.1782 20.7639C29.7751 20.7639 29.4637 20.4451 29.4637 20.0442ZM36.7452 23.7798C36.2781 23.9712 35.811 24.135 35.3622 24.1533C34.6661 24.1897 33.9059 23.9072 33.4938 23.561C32.8527 23.0234 32.3947 22.7229 32.2023 21.7844C32.1199 21.3835 32.1656 20.7639 32.239 20.4087C32.4038 19.6433 32.2206 19.1514 31.6803 18.7048C31.2406 18.3403 30.6819 18.2402 30.0682 18.2402C29.8392 18.2402 29.6287 18.1399 29.4729 18.0579C29.2164 17.9304 29.0059 17.6116 29.2073 17.2197C29.2714 17.0923 29.5829 16.7825 29.6561 16.7278C30.4896 16.2539 31.4513 16.4089 32.3397 16.7642C33.1641 17.1013 33.7869 17.7209 34.6844 18.5955C35.6003 19.6523 35.7651 19.9441 36.2872 20.7366C36.6995 21.3562 37.075 21.9939 37.3314 22.7229C37.4871 23.1785 37.2856 23.552 36.7452 23.7798Z"
                            ></path>
                        </svg>
                        Показать сгенерированный конспект
                    </button>
                    <Modal title="Сгенерированный конспект" bind:open={scrollingModal} class="text-black" size="xl">
                        {@html summary}
                    </Modal>
                {:catch error}
                    <p class="error">{error.message || 'Произошла ошибка при генерации конспекта'}</p>
                {/await}
            {:else}
                <button
                        class="inline-flex items-center px-4 py-2 ml-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors"
                        onclick={generateSummary}
                >
                    <svg
                            class="w-5 h-5 mr-2"
                            fill="none"
                            stroke="currentColor"
                            viewBox="0 0 57 42"
                            xmlns="http://www.w3.org/2000/svg"
                    >
                        <path
                                fill="#4D6BFE"
                                d="M55.6128 3.47122C55.0175 3.17947 54.7611 3.73538 54.413 4.01785C54.2939 4.10892 54.1932 4.22732 54.0924 4.3367C53.2223 5.2659 52.2057 5.87649 50.8776 5.8035C48.9359 5.69412 47.2781 6.30472 45.8126 7.78982C45.5012 5.95853 44.4663 4.86502 42.8909 4.1636C42.0667 3.7991 41.2332 3.4346 40.6561 2.64188C40.2532 2.07718 40.1432 1.44852 39.9418 0.828889C39.8135 0.455354 39.6853 0.0725423 39.2548 0.00882154C38.7877 -0.0639323 38.6045 0.32767 38.4213 0.655795C37.6886 1.99515 37.4047 3.47122 37.4321 4.96536C37.4962 8.32742 38.9159 11.0059 41.7369 12.9102C42.0575 13.1289 42.1399 13.3474 42.0392 13.6665C41.8468 14.3225 41.6178 14.9602 41.4164 15.6162C41.2881 16.0354 41.0957 16.1265 40.647 15.9441C39.0991 15.2974 37.7618 14.3406 36.5803 13.1836C34.5745 11.2429 32.761 9.10184 30.4988 7.42532C29.9675 7.03348 29.4363 6.66922 28.8867 6.32278C26.5786 4.08157 29.189 2.241 29.7935 2.02249C30.4254 1.79471 30.0133 1.01102 27.9708 1.02029C25.9283 1.02933 24.0599 1.71268 21.6786 2.62381C21.3306 2.76053 20.9641 2.86063 20.5886 2.94266C18.4271 2.53274 16.1831 2.44144 13.8384 2.70584C9.42368 3.19778 5.89755 5.28421 3.30551 8.84671C0.19138 13.1289 -0.541286 17.9941 0.356297 23.0691C1.29965 28.4172 4.02902 32.8452 8.22385 36.3076C12.5745 39.8972 17.5845 41.6558 23.2997 41.3186C26.771 41.1182 30.6361 40.6536 34.9958 36.9636C36.0948 37.5103 37.2489 37.7288 39.1632 37.8928C40.6378 38.0295 42.0575 37.8201 43.1565 37.5923C44.8784 37.2278 44.7594 35.6333 44.1366 35.3418C39.09 32.9912 40.1981 33.9478 39.1907 33.1733C41.7552 30.1394 45.6204 26.9868 47.1316 16.7732C47.2506 15.9624 47.1499 15.4521 47.1316 14.7961C47.1224 14.3953 47.214 14.2405 47.672 14.1948C48.9359 14.0491 50.1632 13.7029 51.2898 13.0833C54.5596 11.2976 55.8784 8.3638 56.1898 4.84695C56.2357 4.30936 56.1807 3.75345 55.6128 3.47122ZM27.119 35.123C22.2281 31.2783 19.856 30.0117 18.8759 30.0664C17.96 30.1211 18.1249 31.1689 18.3263 31.8523C18.537 32.5264 18.8118 32.9912 19.1964 33.5833C19.462 33.9751 19.6453 34.5581 18.9309 34.9956C17.3555 35.9705 14.6169 34.6675 14.4886 34.6038C11.3014 32.7268 8.63608 30.2485 6.75839 26.8594C4.94492 23.5974 3.89169 20.0989 3.71762 16.3633C3.67185 15.4614 3.93747 15.1423 4.83505 14.9785C6.01657 14.7598 7.23471 14.7141 8.41623 14.8872C13.408 15.6162 17.6577 17.8484 21.2206 21.3835C23.2539 23.397 24.7926 25.8025 26.3772 28.1531C28.0624 30.6494 29.8759 33.0276 32.184 34.9773C32.9991 35.6606 33.6494 36.1799 34.2722 36.5627C32.3947 36.7722 29.2622 36.8179 27.119 35.123ZM29.4637 20.0442C29.4637 19.6433 29.7843 19.3245 30.1874 19.3245C30.2789 19.3245 30.3613 19.3425 30.4346 19.3699C30.5354 19.4065 30.627 19.4612 30.7002 19.543C30.8285 19.6707 30.9017 19.8528 30.9017 20.0442C30.9017 20.4451 30.5812 20.7639 30.1782 20.7639C29.7751 20.7639 29.4637 20.4451 29.4637 20.0442ZM36.7452 23.7798C36.2781 23.9712 35.811 24.135 35.3622 24.1533C34.6661 24.1897 33.9059 23.9072 33.4938 23.561C32.8527 23.0234 32.3947 22.7229 32.2023 21.7844C32.1199 21.3835 32.1656 20.7639 32.239 20.4087C32.4038 19.6433 32.2206 19.1514 31.6803 18.7048C31.2406 18.3403 30.6819 18.2402 30.0682 18.2402C29.8392 18.2402 29.6287 18.1399 29.4729 18.0579C29.2164 17.9304 29.0059 17.6116 29.2073 17.2197C29.2714 17.0923 29.5829 16.7825 29.6561 16.7278C30.4896 16.2539 31.4513 16.4089 32.3397 16.7642C33.1641 17.1013 33.7869 17.7209 34.6844 18.5955C35.6003 19.6523 35.7651 19.9441 36.2872 20.7366C36.6995 21.3562 37.075 21.9939 37.3314 22.7229C37.4871 23.1785 37.2856 23.552 36.7452 23.7798Z"
                        ></path>
                    </svg>
                    Конспектировать с DeepSeek
                </button>
            {/if}
        </div>
    </div>

    <div class="w-full mb-8">
        <Card class="w-full max-w-none bg-[#c7c1bc] dark:bg-gray-800">
            <Heading class="mb-4 text-xl sm:text-2xl text-[#615151] dark:text-white" tag="h3">
                Написать комментарий
            </Heading>
            <form class="w-full">
                <Textarea
                        bind:value={commentText}
                        class="w-full mb-4 text-xs sm:text-sm"
                        placeholder="Расскажите, как вам лекция"
                        rows={2}
                />
                <Button class="w-full sm:w-auto bg-[#FE8A70]" onclick={handleComment}>
                    Опубликовать
                </Button>
            </form>
        </Card>
    </div>

    <div class="w-full">
        <Heading class="mb-4 text-xl sm:text-2xl text-[#fcefe8] dark:text-white" tag="h3"
        >Комментарии
        </Heading
        >
        <div class="space-y-4 w-full">
            {#if comments.length === 0}
                <Heading tag="h4" class="text-base sm:text-lg text-gray-600 dark:text-gray-300"
                >Комментариев пока нет...
                </Heading
                >
            {:else}
                {#each comments as comment (comment.id)}
                    <Card class="w-full max-w-none bg-[#c7c1bc] flex items-start gap-3">
                        <div class="flex flex-col md:flex-row justify-between">
                            <div
                                    class="flex items-center justify-center w-15 h-11 rounded-full bg-[#615151] text-white font-semibold text-xl"
                            >
                                {comment.author.name.charAt(0).toUpperCase()}
                            </div>
                            <Heading
                                    tag="h2"
                                    class="text-base sm:text-lg font-semibold text-[#615151] dark:text-gray-200 ml-4 md:ml-6 mt-2"
                            >
                                {comment.author.name}
                            </Heading>
                        </div>
                        <div class="ml-16 md:ml-17">
                            <P class="text-base sm:text-lg text-[#615151] dark:text-gray-300">
                                {comment.commentDescription}
                            </P>
                        </div>
                    </Card>
                {/each}
            {/if}
        </div>
    </div>
</div>


<style>
    .lecture-name {
        color: #ffebe0;
        font-size: 2.2rem;
        font-weight: bold;
        margin-bottom: 1rem;
        text-transform: uppercase;
    }
</style>
