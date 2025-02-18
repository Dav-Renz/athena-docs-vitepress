<script setup>
import { onMounted, ref } from 'vue';
import MD from 'markdown-it';

const md = new MD();

const content = ref(md.render('Loading...'));
const url = ref(undefined);

const props = defineProps({
    author: {
        type: String,
    },
    repo: {
        type: String,
    },
    branch: {
        type: String,
    },
});

async function textToByteArray(text) {
    const blob = new Blob([text], { type: 'text/plain' });
    const reader = new FileReader();

    return new Promise((resolve) => {
        reader.onload = (event) => {
            return resolve(new Uint8Array(event.target.result));
        };

        reader.readAsArrayBuffer(blob);
    });
}

async function byteArrayToText(bytes, encoding) {
    const blob = new Blob([bytes], { type: 'application/octet-stream' });
    const reader = new FileReader();

    return new Promise((resolve) => {
        reader.onload = (event) => {
            return resolve(event.target.result);
        };

        if (encoding) {
            reader.readAsText(blob, encoding);
        } else {
            reader.readAsText(blob);
        }
    });
}

onMounted(async () => {
    let res = await fetch(
        `https://raw.githubusercontent.com/${props.author}/${props.repo}/${props.branch}/README.md`
    ).catch(async (err) => {
        console.log(err);
        return undefined;
    });

    if (!res.ok) {
        res = await fetch(
            `https://raw.githubusercontent.com/${props.author}/${props.repo}/${props.branch}/readme.md`
        ).catch((err) => {
            return undefined;
        });
    }

    url.value = `https://github.com/${props.author}/${props.repo}/`;

    if (!res || !res.ok) {
        content.value = md.render(
            '# Failed to Fetch Data\n' +
                '* Ensure that `README.md` is present in the plugin repository.\n' +
                '* Ensure the repository is currently valid.\n' +
                `* [Repo Link](https://github.com/${props.author}/${props.repo}/)`
        );
        return;
    }

    const rawData = await res.text();
    if (!rawData) {
        content.value = md.render(
            '# Failed to Fetch Data\n' +
                '* Ensure that `README.md` is present in the plugin repository.\n' +
                '* Ensure the repository is currently valid.\n' +
                `* [Repo Link](https://github.com/${props.author}/${props.repo}/)`
        );
        return;
    }

    if (rawData.includes('�')) {
        content.value = md.render(
            '# Failed to Fetch Data\n' + '* README IS NOT IN UTF-8 FORMAT. FIX YOUR README FILE.\n'
        );
        return;
    }

    content.value = md.render(rawData);
});
</script>

<template>
    <div class="data">
        <div class="content-data">
            <div v-html="content"></div>
        </div>
        <div class="content-footer" v-if="url">
            <a class="download-btn" :href="url" target="_blank">GitHub (Download)</a>
        </div>
    </div>
</template>

<style>
.data {
    display: flex;
    flex-direction: column;
    padding-top: 48px;
    z-index: 99 !important;
}

.content-footer {
    padding-top: 24px;
}

.content-data {
    margin-bottom: 24px;
}

.content-data img {
    max-width: 400px;
}

.download-btn {
    padding: 12px;
    box-sizing: border-box;
    background: rgba(0, 0, 0, 0.3);
    color: white !important;
    border-radius: 6px;
    min-width: 120px !important;
    max-width: 120px;
}
</style>
