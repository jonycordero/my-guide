# Paginacion y busquedas con Vue y laravel 
Poner este fracmento de codigo para la paginacion y la busqueda


public function listFull(Request $request){
        //dd($request['query']);
        $productos = Producto::orderBy('id', 'ASC')
            ->where('nombre','like','%'.$request['query'].'%')
            ->orwhere('codigo','like','%'.$request['query'].'%')
            ->paginate(5);
        $productos->each(function ($productos) {
            $productos->marca;
            $productos->modelo;
            $productos->categoria;
        });
        return [
            'pagination' => [
                'total'               => $productos->total(),
                'current_page' => $productos->currentPage(),
                'per_page'       => $productos->perPage(),
                'last_page'       => $productos->lastPage(),
                'from'              => $productos->firstItem(),
                'to'                   => $productos->lastItem(),
            ],
            'productos' => $productos
        ];
    }
---------------------------------------------------------------------------------------------------------------
2. completar el listado de los items
------------------------------------------------------------------------------------------------------------
this.listFull(1);

--------------------------------------------------------
// Declarar los atributos de pagination y offset
--------------------------------------------------------
pagination: {
            total: 0,
            current_page: 0,
            per_page: 0,
            last_page: 0,
            from: 0,
            to: 1
        },
offset : 4,
query:'',
loadbusqueda:false,
------------------------------------------------------------------
Modificamos las peticiones get para enviar como parametro el numero de pagina y la busqueda
------------------------------------------------------------------------------------------------
this.loadbusqueda = true;
axios.get(this.url+'/listFull?page='+page+'&query='+this.query)

this.productos = response.data.productos.data;
this.pagination  = response.data.pagination;
this.loadbusqueda = false;

-------------------------------------------------------------------------------------------------
Agregamos cmputed
---------------------------------------------------------------------------------------------------
    computed: {
        isActived: function() {
            return this.pagination.current_page;
        },
        pagesNumber: function() {
            if(!this.pagination.to){
                return [];
            }

            var from = this.pagination.current_page - this.offset;
            if(from < 1){
                from = 1;
            }

            var to = from + (this.offset * 2);
            if(to >= this.pagination.last_page){
                to = this.pagination.last_page;
            }

            var pagesArray = [];
            while(from <= to){
                pagesArray.push(from);
                from++;
            }
            return pagesArray;
        }
    },
--------------------------------------------------------------------------------------------------------
Agregamos el metodo para cambiar depagina con el paginador
--------------------------------------------------------------------------------------------------------
changePage: function(page) {
            this.pagination.current_page = page;
            this.listFull(page); // ojo tener en cuenta tu metodo para listar datos
        }
--------------------------------------------------------------------------------------------------------
<div class="input-group">
     <span v-if="loadbusqueda" class="input-group-addon"><i class="fa fa-spinner fa-spin"></i></span>
     <span v-else class="input-group-addon">Buscar</span>
     <input v-on:keyup.enter="listFull()" v-model="query" type="text" class="form-control" placeholder="Codigo, producto, marca, categoria">
</div>
<br>

-----------------------------------------------------------------------------------------------------------------
<!-- /.box-body -->
            <div class="box-footer">
                <nav>
                    <ul class="pagination">
                        <li v-if="pagination.current_page > 1">
                            <a href="#" @click.prevent="changePage(pagination.current_page - 1)">
                                <span>Atras</span>
                            </a>
                        </li>
                        <li v-for="page in pagesNumber" v-bind:class="[ page == isActived ? 'active' : '']">
                            <a href="#" @click.prevent="changePage(page)">
                                @{{ page }}
                            </a>
                        </li>
                        <li v-if="pagination.current_page < pagination.last_page">
                            <a href="#" @click.prevent="changePage(pagination.current_page + 1)">
                                <span>Siguiente</span>
                            </a>
                        </li>
                    </ul>
                </nav>
                <div>
                    <p>viendo @{{ pagination.from }} a @{{ pagination.to }} de @{{ pagination.total }} resultados</p>
                </div>
            </div>
