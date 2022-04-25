<template>
    <div class="app-container">
      <div class="mt-5">
        <div class="mb-4">

          <div class="d-flex justify-content-center mb-4 ml-1">           
            <input v-for="item in letras_input" :key="item.index"
              type="text"  maxlength="1" 
              class="letter-wrapper rounded d-flex justify-content-center align-items-center mr-2"
              v-model="letras_input[item.index].value"
              @keypress="mudarLetra($event, item.index)"
              v-bind:class="[letras_input[item.index].state]"
              @dblclick="toggleClassInput(item.index)"
              @keydown="toggleClassThroughArrowKey($event, item.index)"
            >                        
          </div>

          <div class="d-flex justify-content-center mb-4">
              <button 
                class="mr-2 rounded "
                @click="resetar()" >
                <span>Resetar</span>
              </button>
              <button
                class="mr-2 rounded "
                @click="procurar()"
                ><span>Procurar</span>
              </button>
          </div>

          <div class="divider"/>

          <div class="px-lg-5 px-2">
            <h5 class="text-center m-4">Letras Inexistentes</h5>
            <div  v-for="linha in listaDeLetrasTeclado" :key="linha.id_linha"
              class="d-flex justify-content-center mb-2"
            >
              <button v-for="tecla in linha" :key="tecla.index"
                  class="keyboard-button rounded me-2 letter-button mr-2"
                  type="button" 
                  v-bind:class="[listaDeLetras[tecla.index].state]"
                  @click="toggleClassKeyboardButton(tecla.index)"
              >
                <span>{{ tecla.value }}</span>
              </button> 
            </div>
          </div>

        </div> 
      </div>

      <!--
      <div v-if="listaDePalavrasEncontradas.length || VerificaSeProcuraInvalida || (!VerificaSeProcuraInvalida && !listaDePalavrasEncontradas.length && verificaSeHaPesquisa)" > 
            <div class="divider"/>
      </div>
      -->
      
      <div class="divider"/>

          <!--
          <div v-for="palavra in listaDePalavrasEncontradas" :key="palavra.id"> 
            <div class="word mr-2">
                {{ palavra }} <br>
            </div> 
          </div>

          <div v-if="listaDePalavrasEncontradas.length > 0 "> 
            <p class="text-center font-weight-normal">

              {{ listaDePalavrasEncontradas.join(', ') }} 
            </p>            
          </div> 
          -->
          
          

      <div class="m-4 word">            
        <div v-if="VerificaSeProcuraInvalida">           
              <h5>Sem Resultados</h5>  
              <p class="m-4">Letra procurada não pode ser igual a letra Inexistentes.</p>  
        </div> 
        <div v-else-if="!VerificaSeProcuraInvalida && !listaDePalavrasEncontradas.length  && verificaSeHaPesquisa">           
              <h5>Sem Resultados</h5>  
              <p class="m-4">Não há resultados para esta combinação.</p>  
        </div> 
        <div v-else-if="listaDePalavrasEncontradas.length > 0 && listaDePalavrasEncontradas.length == 1" >
              <h5>Palavra Encontrada</h5>
        </div>
        <div v-else-if="listaDePalavrasEncontradas.length > 1" >
              <h5>Palavras Encontradas</h5>
        </div>
      </div>  
            
      <div class="found-words-box d-flex justify-content-center overflow-auto word" > 
        <div class="container">   
            <div class="row justify-content-center" v-for="i in Math.ceil(listaDePalavrasEncontradas.length / 5) " :key="i.id">
              <p  class="col mb-1" v-for="palavra in listaDePalavrasEncontradas.slice((i - 1 ) * 5, i * 5)" :key="palavra.id">
                <span>{{palavra}}</span>
              </p>           
            </div>        
        </div>
      </div>      

    </div>
</template>

<script>
export default {
  name: 'App', 
  data (){
    return{
      word_size: 5,
      letras_input:  [],    // array criado dinamicamente
      listaDePalavras: [],
      listaDePalavrasEncontradas: [],
      VerificaSeProcuraInvalida: false, 
      verificaSeHaPesquisa: false,

      //listaDeLetras: ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z'],     
      //listaDeLetras: ['Q','W','E','R','T','Y','U','I','O','P','A','S','D','F','G','H','J','K','L','Z','X','C','V','B','N','M'],    
      /*
      listaDeLetras: [{value:'Q', state:'none'},{value:'W', state:'none'},{value:'E', state:'none'},{value:'R', state:'none'},{value:'T', state:'none'},{value:'Y', state:'none'},{value:'U', state:'none'},{value:'I', state:'none'},{value:'O', state:'none'},{value:'P', state:'none'},
        {value:'A', state:'none'},{value:'S', state:'none'},{value:'D', state:'none'},{value:'F', state:'none'},{value:'G', state:'none'},{value:'H', state:'none'},{value:'J', state:'none'},{value:'K', state:'none'},{value:'L', state:'none'},
        {value:'Z', state:'none'},{value:'X', state:'none'},{value:'C', state:'none'},{value:'V', state:'none'},{value:'B', state:'none'},{value:'N', state:'none'},{value:'M', state:'none'}] ,
      */
      alfabeto: 'QWERTYUIOPASDFGHJKLZXCVBNM'.split(''),
      listaDeLetras: [],
      listaDeLetrasTeclado: [],  // array 'listaDeLetras' adaptado para formato de teclado
    
      //['QWERTYUIOP','ASDFGHJKL','ZXCVBNM',].map(str => str.split(''))

      letras_input_estado_inicial:[],
      listaDeLetras_estado_inicial: []
    }
  },
  created() {
      fetch(`https://rodrigosv91.github.io/Data/words.json`)  // busca todas palavras possiveis
        .then(res => {
          return res.json();
       }).then(this.setWordList),
       
      this.cria_lista_de_inputs(),    // cria array que gerara input para entrada de letras 
      this.cria_lista_de_letras(),    // cria array de letras com 'state' e 'index'
      this.cria_lista_de_letras_teclado()  // cria array que gerara teclado apartir de letras
  },
  methods:{
    setWordList(wordList){
      this.listaDePalavras = wordList;
    },   
    cria_lista_de_inputs(){
        var array_letras_input = []; 
        for(let i = 0; i < this.word_size; i++){
          array_letras_input.push( {'index': i, 'value': "", 'state': "right"});          
        }
        this.letras_input = array_letras_input;
        return array_letras_input;
    },
    cria_lista_de_letras(){
      var array_letras = []; 
      for(let i = 0; i < this.alfabeto.length; i++){
          array_letras.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});          
      }
      this.listaDeLetras = array_letras;
      return array_letras;
    },
    cria_lista_de_letras_teclado(){
      var linha1 = []; 
      var linha2 = []; 
      var linha3 = []; 

      for(let i = 0; i < this.alfabeto.length; i++){
          if(i<10){
            linha1.push( {'index': i, 'value': this.alfabeto[i], 'state': "wrong"});   
          }else
          if(i>=10 && i<19 ){
            linha2.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});   
          }else
          if(i>=19){
            linha3.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});   
          }      
      }
      this.listaDeLetrasTeclado.push(linha1);
      this.listaDeLetrasTeclado.push(linha2);
      this.listaDeLetrasTeclado.push(linha3);
    },
    // muda estado de letra em teclado
    toggleClassKeyboardButton(index){
      /*
      if(this.listaDeLetras[index].state == "wrong"){
        this.listaDeLetras[index].state = "none";
      }else if(this.listaDeLetras[index].state == "none"){
        this.listaDeLetras[index].state = "wrong";
      } */
      this.listaDeLetras[index].state = this.listaDeLetras[index].state == "none" ? "wrong" : "none";
    },
    // muda estado de letra em input
    toggleClassInput(n){
      /*
      if(this.letras_input[n].state == "right"){
        this.letras_input[n].state = "displaced";
      }else if(this.letras_input[n].state == "displaced"){
        this.letras_input[n].state = "right";
      } */
      this.letras_input[n].state = this.letras_input[n].state == "right" ? "displaced" : "right";
    },
    //muda estado de letra em input via Key Arrow
    toggleClassThroughArrowKey(event, InputIndex){
      if(event.key == "ArrowUp" || event.key == "ArrowDown"){
        this.toggleClassInput(InputIndex);
        this.procurar();
      }
      else
      if(event.key == "ArrowRight"){
        event.target.nextElementSibling.focus();
        this.procurar();
      }
      else
      if(event.key == "ArrowLeft"){
        event.target.previousElementSibling.focus();
        this.procurar();
      }
    },
    
    mudarLetra(e, n){     
      if(e.key == "Enter"){ 
        /*
        this.listaDePalavrasEncontradas = [];

        var newArray = [];
        var existeLetraBusca = false;
        var existeLetraErrada = false;

        //verifica se existe letra a ser buscada em input
        for (var i = 0; i < this.letras_input.length; ++i) {
          if(this.letras_input[i].value != ''){
            existeLetraBusca = true;
            break;
          }
        }

        //verifica se existe alguma letra errada a ser filtrada das palavras
        let letrasErradas = this.listaDeLetras.filter(function checkWrong(obj) {
          return obj.state == 'wrong';
        });
        existeLetraErrada = letrasErradas.length > 0 ? true : false;    //teste
  
        //carrega lista de palavras se existe alguma letra a ser buscada (input) ou filtrada (letra errada)
        if(existeLetraBusca || existeLetraErrada){
          this.listaDePalavrasEncontradas = this.listaDePalavras;
        }
        
        //verifica se a letra existe na posição "N" da palavra 
        this.letras_input.forEach( item => {
          if(item.value != '' && item.state == 'right'){
            newArray = [];
            this.listaDePalavrasEncontradas.forEach(palavraTestada => {
                if(palavraTestada[item.index] == item.value.toUpperCase())
                  newArray.push(palavraTestada);
            });
            this.listaDePalavrasEncontradas = newArray;
          }
        });    
        
        //verifica se existe a letra na palavra        
        this.letras_input.forEach( item => {
          if(item.value != '' && item.state == 'displaced'){
          newArray = [];
          this.listaDePalavrasEncontradas.forEach(palavra => {
              if(palavra.includes(item.value.toUpperCase()) && palavra[item.index] != item.value.toUpperCase() ){
                newArray.push(palavra);
              }
          });
          this.listaDePalavrasEncontradas = newArray;
          }
        });

        //exclue da lista palavras que contem alguma das letras excluidas (wrong)        
        if(existeLetraErrada){
          letrasErradas.forEach(l => {
            newArray = []; 
            this.listaDePalavrasEncontradas.forEach(palavra => {
              if(!palavra.includes(l.value)){
                newArray.push(palavra);
              }
            });
            this.listaDePalavrasEncontradas = newArray;
          });      
        }
        */
        this.procurar();
      } 

      //muda valor da letra em Data
      let ltr = e.key;

      function exists(arr, search) {
          return arr.some(palavra => palavra.includes(search)); // metodo "some" consegue verificar em array multidimensional
      }                                                         // https://stackoverflow.com/questions/52103644/includes-in-multidimensional-array
      if (ltr.length === 1 && exists(this.alfabeto, ltr.toUpperCase() ) ) {  
        this.letras_input[n].value = ltr;
      }else {
        e.preventDefault();
      }
    },
 
    procurar(){
      this.listaDePalavrasEncontradas = [];

      var newArray = [];
      var existeLetraBusca = false;
      var existeLetraErrada = false;
      this.VerificaSeProcuraInvalida = false;

      //verifica se existe letra a ser buscada em input
      for (var i = 0; i < this.letras_input.length; ++i) {
        if(this.letras_input[i].value != ''){
          existeLetraBusca = true;
          break;
        }
      }

      //verifica se existe alguma letra errada a ser filtrada das palavras
      let letrasErradas = this.listaDeLetras.filter(function checkWrong(obj) {
        return obj.state == 'wrong';
      });
      existeLetraErrada = letrasErradas.length > 0 ? true : false;

      //carrega lista de palavras se existe alguma letra a ser buscada (input) ou filtrada (letra errada)
      if(existeLetraBusca || existeLetraErrada){
        this.listaDePalavrasEncontradas = this.listaDePalavras;
      }
      
      //verifica se a letra existe na posição "N" da palavra 
      this.letras_input.forEach( item => {
        if(item.value != '' && item.state == 'right'){
          newArray = [];
          this.listaDePalavrasEncontradas.forEach(palavraTestada => {
              if(palavraTestada[item.index] == item.value.toUpperCase())
                newArray.push(palavraTestada);
          });
          this.listaDePalavrasEncontradas = newArray;
        }
      });    
      
      //verifica se existe a letra na palavra        
      this.letras_input.forEach( item => {
        if(item.value != '' && item.state == 'displaced'){
        newArray = [];
        this.listaDePalavrasEncontradas.forEach(palavra => {
            if(palavra.includes(item.value.toUpperCase()) && palavra[item.index] != item.value.toUpperCase() ){
              newArray.push(palavra);
            }
        });
        this.listaDePalavrasEncontradas = newArray;
        }
      });

      //exclue da lista palavras que contem alguma das letras excluidas (wrong)        
      if(existeLetraErrada){
        letrasErradas.forEach(letra_errada => {
          newArray = []; 
          this.listaDePalavrasEncontradas.forEach(palavra => {
            if(!palavra.includes(letra_errada.value)){
              newArray.push(palavra);
            }
          });
          this.listaDePalavrasEncontradas = newArray;
        });      
      }

      
      this.letras_input.forEach(letra_input => {
          letrasErradas.forEach(letra_errada => {
            //alert(letra_input.value.toUpperCase() == letra_errada.value.toUpperCase())
            if(letra_input.value.toUpperCase() == letra_errada.value.toUpperCase()){
                this.VerificaSeProcuraInvalida = true;
            }
          });    
      });

      this.verificaSeHaPesquisa = true
      
      
    },

    resetar(){
      this.letras_input = this.cria_lista_de_inputs();
      this.listaDeLetras = this.cria_lista_de_letras();
      this.listaDePalavrasEncontradas = [];
      this.VerificaSeProcuraInvalida = false;
      this.verificaSeHaPesquisa = false;
    }
  }
}
</script>

<style>
.app-container {
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
}
.app-name {
  font-family: 'Acme', sans-serif;
  font-size: 270%;
}
.letter-wrapper {
  border: 2px solid #252525;
  width: 48px;
  height: 40px;

  font-size: 20px;
  font-weight: bolder;
}
input { 
    text-align: center; 
    text-transform: uppercase;
    /*background-color: rgb(81, 179, 110);   */
}
.right{
  background-color: #51b36e;
  color: #f4f3f1;
}
.displaced {
  background-color: #c79c2e;
  color: #f4f3f1 !important;
}
.wrong {
  background-color: #943e3c;
  color: #f4f3f1;
}

.keyboard-button {
  border: solid 1px #252525;
  font-size: 16px;
}
.letter-button {
  width: 8%;
  height: 40px;
}
.found-words-box{
  max-width: 450px;
  max-height: 250px;
  margin-left: auto;
  margin-right: auto;
}
.col{
  max-width: 20%;
}

.word{
  text-align: center;
}

.divider {
  flex-grow: 1;
  border-bottom: 1px solid black;
  margin: 5px
}
</style>
