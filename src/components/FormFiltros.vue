<template>
    <div>
        <v-list-item>
            <v-list-item-content>
                <v-list-item-title class="text-h6 cyan--text">
                    Buscar Estações
                </v-list-item-title>
                <v-list-item-subtitle>
                    Selecione os filtros
                </v-list-item-subtitle>
            </v-list-item-content>
        </v-list-item>

      <v-divider class="m-sm"></v-divider>
        <v-row no-gutters>
            <v-col cols="12" md="12" class="px-4 pt-4">
                <v-select
                    dense
                    small-chips
                    deletable-chips
                    v-model="form.tiposEstacao"
                    :items="listaTiposEstacoes"
                    return-object
                    :menu-props="{ maxHeight: '400' }"
                    outlined
                    item-text="name"
                    label="Tipos de estação"
                    multiple
                    @input="estacoesLiberadas()"
                >
                    <template v-slot:selection="{ item, index }">
                        <v-chip class="ma-1" small v-if="index < 3">
                            <span>{{ item.name }}</span>
                        </v-chip>
                        <span
                            v-if="index === 3"
                            class="px-2 grey--text text-caption"
                        >
                            + Outros
                        </span>
                    </template>

                    <template v-slot:prepend-item>
                        <v-list-item
                            ripple
                            @mousedown.prevent
                            @click="selectedAll('tiposEstacao', listaTiposEstacoes), estacoesLiberadas()"
                        >
                            <v-list-item-action>
                                <v-icon v-if="form.tiposEstacao == 0">
                                    mdi-checkbox-blank-outline
                                </v-icon>
                                <v-icon v-else color="indigo darken-4">
                                    {{ form.tiposEstacao.length === listaTiposEstacoes.length 
                                        ? 'mdi-close-box'
                                        : 'mdi-minus-box' 
                                    }}
                                </v-icon>
                            </v-list-item-action>
                            <v-list-item-content>
                                <v-list-item-title>
                                    Selecionar todos
                                </v-list-item-title>
                            </v-list-item-content>
                        </v-list-item>
                        <v-divider class="mt-2"></v-divider>
                    </template>
                </v-select>
            </v-col>
        </v-row>
       <v-row no-gutters>
            <v-col cols="12" md="12" class="px-4 pt-4">
                <v-autocomplete
                    :disabled="!listaEstacoesfiltro.length > 0"
                    dense
                    small-chips
                    deletable-chips
                    v-model="form.estacaoes"
                    :items="listaEstacoesfiltro"
                    :menu-props="{ maxHeight: '400' }"
                    outlined
                    item-text="name"
                    return-object
                    label="Tipos de estação"
                    multiple
                >
                    <template v-slot:selection="{ item, index }">
                        <v-chip class="ma-1" small v-if="index < 3">
                            <span>{{ item.name }}</span>
                        </v-chip>
                        <span
                            v-if="index === 3"
                            class="px-2 grey--text text-caption"
                        >
                            + Outros
                        </span>
                    </template>

                    <template v-slot:prepend-item>
                        <v-list-item
                            ripple
                            @mousedown.prevent
                            @click="selectedAll('estacaoes', listaEstacoesfiltro)"
                        >
                            <v-list-item-action>
                                <v-icon v-if="form.estacaoes == 0">
                                    mdi-checkbox-blank-outline
                                </v-icon>
                                <v-icon v-else color="indigo darken-4">
                                    {{ form.estacaoes.length === listaEstacoesfiltro.length 
                                        ? 'mdi-close-box'
                                        : 'mdi-minus-box' 
                                    }}
                                </v-icon>
                            </v-list-item-action>
                            <v-list-item-content>
                                <v-list-item-title>
                                    Selecionar todos
                                </v-list-item-title>
                            </v-list-item-content>
                        </v-list-item>
                        <v-divider class="mt-2"></v-divider>
                    </template>
                </v-autocomplete>
            </v-col>
        </v-row>
        <v-row no-gutters class="justify-end">
            <v-col cols="12" md="12" class="text-right px-4">
                <v-btn
                    :disabled="form.estacaoes.length < 1"
                    class="white--text text-uppercase cyan"
                    @click="setFiltros()"
                >
                    Consultar
                <v-icon
                    rigth
                    dark
                >
                    mdi-refresh
                </v-icon>
                </v-btn>
            </v-col>
        </v-row>
    </div>
</template>

<script>
    const urlStationType = 'http://localhost:3000/station_type';
    const urlStations = 'http://localhost:3000/stations';

    export default {
        name: 'FormFiltros',
        data: () => ({
            form: {
                tiposEstacao: [],
                estacaoes: [],
            },
            listaTiposEstacoes: [],
            listaEstacoes: [],
            listaEstacoesfiltro: [],
            filtroEstacoes: []
        }),
        created(){
            this.getListaTiposEstacoes()
            this.getListaEstacoes()
        },
        methods: {
            estacoesLiberadas(){
                this.form.estacaoes = [];
                var lista = [];
                Object.keys(this.form.tiposEstacao).forEach(item =>{
                    let id = this.form.tiposEstacao[item].id
                    this.listaEstacoes.map(item => {
                        if(item.station_type_id == id){
                            lista.push(item)
                        }
                    })
                })
                this.listaEstacoesfiltro = lista
            },
            getListaTiposEstacoes(){
                fetch(urlStationType).then(res =>{
                    return res.json();
                }).then(data =>{
                    this.listaTiposEstacoes = data
                })
            },
            getListaEstacoes(){
                fetch(urlStations).then(res =>{
                    return res.json();
                }).then(data =>{
                    this.listaEstacoes = data
                })
            },
            selectedAll(select, lista) {
                if (this.form[select].length === lista.length) {
                    this.form[select] = []
                } else {
                    this.form[select] = lista
                }
            },
            setFiltros(){
                this.$emit('filtroMarkers', this.form.estacaoes)
            }
        },
    }
</script>