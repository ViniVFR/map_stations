<template>
    <v-card>
        <v-card-title class="cyan--text text-h5 grey lighten-2">
            Dados da Estação
        </v-card-title>
        <v-card-text class="pa-4" style="max-height:70vh; overflow: auto;">
            <v-row no-gutters>
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.id"
                        label="identificador"
                        readonly
                    ></v-text-field>
                </v-col>
                <v-col cols="12" md="8" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.name"
                        label="Name"
                        readonly
                    ></v-text-field>
                </v-col>
            </v-row>
            <v-row no-gutters class="justify-end">
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.latitude"
                        label="Latitude"
                        readonly
                    ></v-text-field>
                </v-col>
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.longitude"
                        label="Longitude"
                        readonly
                    ></v-text-field>
                </v-col>
            </v-row>
            <v-row no-gutters>
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.elevation_meters"
                        label="Elevacao (m²)"
                        readonly
                    ></v-text-field>
                </v-col>
                <!--
                * TODO
                * Colocar um datepicker nos text-field das datas
                *   
                -->
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        v-model="body.operation_start_date"
                        label="Inicio de Operação"
                        readonly
                    ></v-text-field>
                </v-col>
                <v-col cols="12" md="4" class="px-2">
                    <v-text-field
                        dense
                        outlined
                        return-object
                        v-model="body.operation_end_date"
                        label="Fim de Operação"
                        readonly
                    ></v-text-field>
                </v-col>
            </v-row>
            <v-row no-gutters>
                <v-col cols="12" class="px-2">
                    <v-autocomplete
                        dense
                        outlined
                        readonly
                        item-text="name"
                        value="2"
                        return-object
                        :items="tiposEstacoes"
                        v-model="body.station_type_id"
                        label="Tipo de Estação"
                    ></v-autocomplete>
                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>

<script>
    const urlStationType = 'http://localhost:3000/station_type';
    
    export default {
        name: 'PopUpMarker',
        props: {
            dados: Object
        },
        data: () => ({
            body: {
                id: null,
                name: '',
                latitude: '',
                longitude: '',
                elevation_meters: '',
                operation_start_date: '',
                operation_end_date: '',
                station_type_id: ''
            },
            tiposEstacoes: []
        }),
        watch:{
            dados(){
                this.setDados()
            }
        },
        async created(){
            await this.getListaTiposEstacoes()
            this.setDados()
        },
        methods: {
            async getListaTiposEstacoes(){
                await fetch(urlStationType).then(res =>{
                    return res.json();
                }).then(data =>{
                    this.tiposEstacoes = data
                })
            },
            setDados(){
                let dadosRef = Object.assign({}, this.dados)

                this.body = dadosRef;
                this.body.station_type_id = this.getTipoEstacao(dadosRef.station_type_id);
                this.body.operation_start_date = this.formatarData(dadosRef.operation_start_date);
                this.body.operation_end_date = this.formatarData(dadosRef.operation_end_date);
            },
            getTipoEstacao(tipoId){
                if(!tipoId) return '';
                let index = this.tiposEstacoes.findIndex(key => key.id == tipoId)

                if (index == -1) return tipoId;
                return Object.assign({},this.tiposEstacoes[index])
            },
            formatarData(data) {
                if(!data) return '';
                if(!data.includes('-')) return data

                var [ano, mes, dia] = data.split('-')
                return `${dia}/${mes}/${ano}`
            },
        }
    }
</script>
