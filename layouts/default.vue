<template>
    <div>
        <b-navbar toggleable="md" type="dark" variant="info" sticky>
            <b-navbar-brand to="/">Graph The Shit</b-navbar-brand>
            <b-nav is-nav-bar class="ml-auto">
                <b-nav-item exact to="/">Home</b-nav-item>
                <b-nav-item to="/graphs">Graphs</b-nav-item>
            </b-nav>
        </b-navbar>
        <div class="padding-from-nav">
            <nuxt/>
        </div>
    </div>
</template>

<style scoped>
    .padding-from-nav {
        padding-top: 20px;
    }
</style>