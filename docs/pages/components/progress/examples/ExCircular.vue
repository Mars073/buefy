<template>
    <section>
        <b-progress :rounded="false" :circular="true" format="percent" :max="80">
            <template #bar>
                <b-progress-bar :value="10" show-value></b-progress-bar>
                <b-progress-bar :value="20" type="is-primary" show-value></b-progress-bar>
                <b-progress-bar :value="30" type="is-warning" show-value></b-progress-bar>
            </template>
        </b-progress>
        <b-progress :rounded="true" :circular="true" :circle-start="-136" :circle-end="136">
            <template #bar>
                <b-progress-bar :value="10" show-value></b-progress-bar>
                <b-progress-bar :value="20" type="is-primary" show-value></b-progress-bar>
                <b-progress-bar :value="30" type="is-warning" show-value></b-progress-bar>
            </template>
        </b-progress>
    </section>
</template>
