<template>
    <div>
        <b-nav tabs>
            <b-nav-item disabled><b>Fancy Quiz App</b></b-nav-item>
            <b-nav-item disabled>Count: {{currentQuestionIndex + 1}} / 10</b-nav-item>
        </b-nav>
    </div>
</template>

<script>
export default {
    props: {
        currentQuestionIndex: Number
    }
}
</script>