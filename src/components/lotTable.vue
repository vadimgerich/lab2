// таблична форма відображення лотів
<template>
    <div>
        <p v-if="lotList.length==0" class="alert">
            Лоти відсутні
        </p>
        
        <table v-if="lotList.length>0">
            <tr>
                <th>#</th>
                <th v-on:click="sort('title')">  Назва </th>
                <th v-on:click="sort('startData')"> Дата початку торгу </th>
                <th v-on:click="sort('endData')"> Дата кінця торгу </th>
                <th v-on:click="sort('startPrice')"> Початкова ціна </th>
                <th v-on:click="sort('endPrice')"> Кінцева ціна </th>
                <th></th>
            </tr>
            <lotTableRow v-for="(lot,index) in lotList" 
                v-bind:key="lot._id" 
                v-bind="{lot,index}"
                @remove="remove"
                @update="update" 
            >             
            </lotTableRow>
        </table>
    </div> 
</template>

<script>
import lotTableRow from "./lotTableRow";

export default {
    name:"lotTable",
    props:{
        lotList:Array,
    },
    data(){
        return{
           lotss: this.lotList
        }
    },
    components:{
        lotTableRow
    },
    methods:{
        //сортування по деякому полю 
        sort(field){
           this.lotList.sort((b1,b2)=> b1[field]>=b2[field]?1:-1);
        },
        //вилучити лот
        remove(index){
            //генруємо подію, вилучення здійснить батьківський компонент
            this.$emit("remove",index);
        },  
        //оновити лот
        update(index){
            //генруємо подію, оеовлення здійснить батьківський компонент
            this.$emit("update",index);
        }
    }
}
</script>

<style scoped>
    .alert{
        background: yellow;
        color: crimson;
    }

    table, table td{
        border-collapse: collapse;
        border: 1px solid black;
    }
</style>