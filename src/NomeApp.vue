<template>
    <span   v-for="(letra, index) in app_name" :key="index"
            v-bind:class="letra.className"
    >
            {{ letra.value }}
    </span>
</template>

<script>
export default {
    props: ['app_name_prop'],
    data (){
        return{
            app_name: this.app_name_prop.toUpperCase(),
            app_name_classes:['letter-green','letter-yellow','letter-red','letter-green','letter-yellow','letter-red']
        }
    },
    created() { 
        const shuffledIndexes = this.shuffleArray(Array.from(Array(this.app_name.length).keys()));

        this.app_name = this.app_name.split('').map((letter, index) => {
            let className = '';
            className = this.app_name_classes[shuffledIndexes[index]];  
            return {'value': letter, 'className': className};
        });
    },
    methods:{
        shuffleArray(array){
            let shuffled = [...array];
            
            let currentIndex = shuffled.length
            let randomIndex;

            while (currentIndex !== 0) {
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;

                [shuffled[currentIndex], shuffled[randomIndex]] = [
                shuffled[randomIndex], shuffled[currentIndex]];
            }

            return shuffled;
        }
    }
}
</script>

<style>
</style>