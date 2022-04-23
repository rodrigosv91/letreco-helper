<template>
    <div class="app-container">
      <div class="mt-5">
        <div class="mb-4">

          <div class="d-flex justify-content-center mb-4">           
            <input v-for="item in letras_input" :key="item.index"
              type="text"  maxlength="1" 
              class="letter-wrapper rounded d-flex justify-content-center align-items-center mr-2"
              v-model="letras_input[item.index].value"
              @keypress="mudaLetra($event, item.index)"
              v-bind:class="[letras_input[item.index].state]"
              @click="toggleClassInput(item.index)"
            >                        
          </div>

          <div class="d-flex justify-content-center mb-4">
              <button class="mr-2"
                @click="resetar()" >
                <span>Resetar</span>
              </button>
              <button
              @click="procurar()"
              ><span>Procurar</span></button>
          </div>

          <div class="px-lg-5 px-2">
            <div  v-for="linha in listaDeLetrasTeclado" :key="linha.id_linha"
              class="d-flex justify-content-center mb-2"
            >
              <button v-for="tecla in linha" :key="tecla.index"
                  class="keyboard-button rounded me-2 letter-button mr-2"
                  type="button" 
                  v-bind:class="[listaDeLetras2[tecla.index].state]"
                  @click="toggleClassKeyboardButton(tecla.index)"
              >
                <span>{{ tecla.value }}</span>
              </button> 
            </div>
          </div>

        </div> 
      </div>

      <div class="mt-5">
        <div class="d-flex justify-content-center" id="foundWords">        
          {{ listaDePalavrasEncontradas }}
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

      //listaDeLetras: ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z'],     
      //listaDeLetras: ['Q','W','E','R','T','Y','U','I','O','P','A','S','D','F','G','H','J','K','L','Z','X','C','V','B','N','M'],    
      /*
      listaDeLetras2: [{value:'Q', state:'none'},{value:'W', state:'none'},{value:'E', state:'none'},{value:'R', state:'none'},{value:'T', state:'none'},{value:'Y', state:'none'},{value:'U', state:'none'},{value:'I', state:'none'},{value:'O', state:'none'},{value:'P', state:'none'},
        {value:'A', state:'none'},{value:'S', state:'none'},{value:'D', state:'none'},{value:'F', state:'none'},{value:'G', state:'none'},{value:'H', state:'none'},{value:'J', state:'none'},{value:'K', state:'none'},{value:'L', state:'none'},
        {value:'Z', state:'none'},{value:'X', state:'none'},{value:'C', state:'none'},{value:'V', state:'none'},{value:'B', state:'none'},{value:'N', state:'none'},{value:'M', state:'none'}] ,
      */
      alfabeto: 'QWERTYUIOPASDFGHJKLZXCVBNM'.split(''),
      listaDeLetras2: [],
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
          array_letras_input.push( {'index': i, 'value': "", 'state': "none"});          
        }
        this.letras_input = array_letras_input;
        return array_letras_input;
    },
    cria_lista_de_letras(){
      var array_letras = []; 
      for(let i = 0; i < this.alfabeto.length; i++){
          array_letras.push( {'index': i, 'value': this.alfabeto[i], 'state': "none"});          
      }
      this.listaDeLetras2 = array_letras;
      return array_letras;
    },
    cria_lista_de_letras_teclado(){
      var linha1 = []; 
      var linha2 = []; 
      var linha3 = []; 

      for(let i = 0; i < this.listaDeLetras2.length; i++){
          if(i<10){
            linha1.push( {'index': i, 'value': this.listaDeLetras2[i].value, 'state': "wrong"});   
          }else
          if(i>=10 && i<19 ){
            linha2.push( {'index': i, 'value': this.listaDeLetras2[i].value, 'state': "none"});   
          }else
          if(i>=19){
            linha3.push( {'index': i, 'value': this.listaDeLetras2[i].value, 'state': "none"});   
          }      
      }
      this.listaDeLetrasTeclado.push(linha1);
      this.listaDeLetrasTeclado.push(linha2);
      this.listaDeLetrasTeclado.push(linha3);
    },
    // muda estado de letra em teclado
    toggleClassKeyboardButton(index){

      if(this.listaDeLetras2[index].state == "wrong"){
        this.listaDeLetras2[index].state = "none";
      }else if(this.listaDeLetras2[index].state == "none"){
        this.listaDeLetras2[index].state = "wrong";
      }else {
        this.listaDeLetras2[index].state = "none";
      }
    },
    // muda estado de letra em input
    toggleClassInput(n){
      if(this.letras_input[n].state == "right"){
        this.letras_input[n].state = "displaced";
      }else if(this.letras_input[n].state == "displaced"){
        this.letras_input[n].state = "right";
      }else {
        this.letras_input[n].state = "right";
      }
      //this.letras_input[n].state = this.letras_input[n].state == "right" ? "displaced" : "right";
    },

    mudaLetra(e, n){     
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
        let letrasErradas = this.listaDeLetras2.filter(function checkWrong(obj) {
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

      //verifica se existe letra a ser buscada em input
      for (var i = 0; i < this.letras_input.length; ++i) {
        if(this.letras_input[i].value != ''){
          existeLetraBusca = true;
          break;
        }
      }

      //verifica se existe alguma letra errada a ser filtrada das palavras
      let letrasErradas = this.listaDeLetras2.filter(function checkWrong(obj) {
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
    },

    resetar(){
      this.letras_input = this.cria_lista_de_inputs();
      this.listaDeLetras2 = this.cria_lista_de_letras();
      this.listaDePalavrasEncontradas = [];
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
</style>
