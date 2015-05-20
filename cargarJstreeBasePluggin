var CargarJstreeBasePluggin = function ($1) {

    // Singleton que se encarga de manejar las consultas AJAX de forma de que se ejecuten en serie y NO en paralelo.
    var ConsultasAjax = new function () {
        var peticionesAjax = [];
        var $ejecutandoPeticion = null;

        this.AgregarPeticion = function (ajaxFunc) {
            peticionesAjax.push(ajaxFunc);
            if ($ejecutandoPeticion == null) {
                this.EjecutarProximaPeticion();
            }
        }

        this.ObtenerPeticion = function () {
            if (peticionesAjax.length > 0) {
                // Obtengo la primer peticion de la lista para procesar
                var $procesar = peticionesAjax[0];
                // Quito la 1er peticion de la lista. Para eso , creo un nuevo array con los valores
                var nuevoArray = [];
                for (var key in peticionesAjax) {
                    if (key > 0) {
                        nuevoArray.push(peticionesAjax[key]);
                    }
                }
                peticionesAjax = nuevoArray;
                return $procesar;
            } else {
                return null;
            }
        }

        this.EjecutarProximaPeticion = function () {
            $ejecutandoPeticion = ConsultasAjax.ObtenerPeticion();
            if ($ejecutandoPeticion != null) {
                var $ajax = $ejecutandoPeticion.call();
                $ajax.then(ConsultasAjax.EjecutarProximaPeticion);
            }
        }
    }

    var ObtenerDatos = function (metodoCallBack, contexto,config) {
        var success = function (data, textStatus, jqXHR) {
            config.respuestaLectura(data);

            if (data.responseOK) {
                metodoCallBack.call(contexto, data.items);
            }           
        }

        var error = function (data, textStatus, jqXHR) {
            alert('Fallo la carga de datos: ' + jqXHR);
        }

        this.raices = function (config, parametros) {
            var url = config.url + config.metodo;

            var ajaxFunc = function () {
                return $1.ajax({
                    type: 'GET',
                    dataType: "json",
                    url: url,
                    data: parametros,
                    cache: false,
                    success: success,
                    error: error
                });
            }

            ConsultasAjax.AgregarPeticion(ajaxFunc);
        }

        this.hijos = function (config,parametros) {

            var url = config.url + config.metodo;

            var ajaxFunc = function () {
                return $1.ajax({
                    type: 'GET',
                    dataType: "json",
                    url: url,
                    data: parametros,
                    cache: false,
                    success: success,
                    error: error
                });
            }

            ConsultasAjax.AgregarPeticion(ajaxFunc);
        }
    }

    var GuardarDatos = function (nodoJsTree, nodoPadreJsTree,config) {
        this.resultado = false;

        this.EjecutarFuncionRespuestaCorrecta = function (data) { }

        var success = function (data, textStatus, jqXHR) {
            config.respuestaEscritura(data);
            this.resultado = data.responseOK;
            if (data.responseOK) {
                this.EjecutarFuncionRespuestaCorrecta(data);
            }
        }

        var error = function (data, textStatus, jqXHR) {
            alert(jqXHR);
        }

        this.EstablecerJerarquia = function (keyNodo, keyNodoPadre,funcionCallBack) {

            var configuracion;

            // Verifico si se trata de una baja o un alta/modificacion
            if (keyNodoPadre != null) {
                configuracion = config.escrituraJerarquia;
            } else {
                configuracion = config.borrarJerarquia;
            }

            var parametros = configuracion.funcionParaGenerarParametros(keyNodo, keyNodoPadre);

            var url = configuracion.url + configuracion.metodo;

            if (funcionCallBack) this.EjecutarFuncionRespuestaCorrecta = funcionCallBack;

            $1.ajax({
                type: 'POST',
                url: url,
                data: parametros,
                dataType: 'json',
                async: false,
                success: success,
                error: error,
                context: this
            });
        }

        this.Mover = function () {
            var confirmado = config.CallBack.PedirConfirmacionActualizarNodo(nodoJsTree, nodoPadreJsTree);
            if (confirmado) {
                this.EstablecerJerarquia(nodoJsTree.data.key, nodoPadreJsTree.data.key);
            }
        }

        this.Agregar = function () {
            var confirmado = config.CallBack.PedirConfirmacionNuevoNodo(nodoJsTree, nodoPadreJsTree);
            if (confirmado) {
                this.EstablecerJerarquia(nodoJsTree.data.key, nodoPadreJsTree.data.key);
            }
        }

        this.EliminarAsincronico = (function (guardarDatos) {
            var funcionEliminarItemArbol;

            var funcionEliminar = function () {
                guardarDatos.EstablecerJerarquia(nodoJsTree.data.key, null, funcionEliminarItemArbol);
            };

            return function (EliminarItemArbol) {
                funcionEliminarItemArbol = EliminarItemArbol;
                config.CallBack.PedirConfirmacionEliminarNodo(nodoJsTree, funcionEliminar);
            }
        })(this);
    }

    this.GetConfigItem = function () {
        var Origen = (function(){
            this.funcionParaGenerarParametros = (function (keyNodo, keyNodoPadre) {
                return {
                    keyNodo: keyNodo,
                    keyNodoPadre: keyNodoPadre
                }
                // Si hacen falta agregar mas parametros, reemplazar el valor de la propiedad this.parametros con la nueva funcion
            });
            this.url = '';
            this.metodo = '';
        });

        var RespuestaServidor = function () {
            return function (data) {
                if (!data.responseOK) {
                    alert(data.exception.message);
                }
            }
        }

        var PreguntaServidor = function () {
            return function (nodoNuevo, nodoPadre) {
                return confirm('Seguro desea mover ' + nodoNuevo.text + ' dentro de ' + nodoPadre.text);
            }
        }

        var PreguntaServidorAsincronica = function () {
            return function (nodo, funcionEliminar) {
                // funcionEliminar()

                var respuesta = confirm("Se ejecutará la acción sobre el nodo " + nodo.text + ". Desea continuar?");
                if (respuesta) {
                    funcionEliminar();
                }
            }
        }

        return new (function() {
            this.origenRaices = new Origen();
            this.origenHijos = new Origen();
            this.escrituraJerarquia = new Origen();
            this.borrarJerarquia = new Origen();
            this.CallBack = {
                PedirConfirmacionNuevoNodo: new PreguntaServidor(),
                PedirConfirmacionActualizarNodo: new PreguntaServidor(),
                PedirConfirmacionEliminarNodo: new PreguntaServidorAsincronica()
            }
            this.Eventos = {
                ArrastrandoNodo: function (nodoOrigen, nodoDestino) {
                    
                },
                FinArrastre: function (nodo) {
                    
                },
                SoltoNodo: function (nodoOrigen, nodoDestino) {
                    
                },
                CopiandoNodo: function (nodoOriginal, nodoNuevo) {

                }
            }
            this.respuestaLectura = new RespuestaServidor();
            this.respuestaEscritura = new RespuestaServidor();
        })();
    }

    var NewjsTreeData = (function(config){
        return (function (obj, cb) {
            var datos = new ObtenerDatos(cb, this,config);
            if (obj.id === '#') {
                var parametros = config.origenRaices.funcionParaGenerarParametros(null, null);
                datos.raices(config.origenRaices, parametros);
            } else {
                var parametros = config.origenHijos.funcionParaGenerarParametros(obj.data.key,null);
                datos.hijos(config.origenHijos, parametros);
            }
        });
    });


    
    $1.fn.jsTreeDnD = (function (config) {
        if (config == undefined) throw new Error("Falta parametros de configuracion");

        var $contenedorParaJsTree = this;

        $1(function () {

            var ContextMenuItems = (function (node) {
                var items = {};

                var deleteAction = function (obj) {
                    var EliminarItemArbol = function (data)
                    {
                        $jsTreeDiv.jstree('delete_node',obj.reference);
                    }

                    var guardar = new GuardarDatos(node, null, config);
                    guardar.EliminarAsincronico(EliminarItemArbol);
                }
                
                if (node.data.contextMenu.delItem.visible) {

                    items.delItem = {
                        label: 'Borrar',
                        icon: '../../imgs/cancel_changes_small.PNG',
                        action: deleteAction
                    };
                }
                return items;
            });

            var $jsTreeDiv = $contenedorParaJsTree
                // create instance
                .jstree({
                    "core": {
                        "multiple": false,
                        "check_callback": function (operation, node, node_parent, node_position, more) {
                            if (operation === 'delete_node') {
                                return true;
                            } else if (operation === 'move_node' || operation === 'copy_node') {
                                if (
                                    node.parent != node_parent.id // Verifico que no se este arrastrando sobre su propio padre
                                    && node.data.drag.enabled // Verifico que permita DRAG
                                    && node_position === 0) { // Prevent siblings
                                    if (more.pos === undefined) {
                                        // Solto el nodo
                                        config.Eventos.SoltoNodo(node, node_parent);

                                        if (node_parent.data.drop.enabled) { // Verifico si el nodo de destino permite drop

                                            var guardar = new GuardarDatos(node, node_parent, config);

                                            if (operation === 'move_node') {
                                                guardar.Mover();
                                            } else if (operation === 'copy_node') {
                                                guardar.Agregar();
                                            }
                                            return guardar.resultado;
                                        } else {
                                            return false;
                                        }
                                    } else {
                                        // Lo está moviendo por encima
                                        if (more.pos === 'i') {
                                            config.Eventos.ArrastrandoNodo(node, node_parent);
                                            return node_parent.data.drop.enabled;      // Verifico si el nodo de destino permite drop                                          
                                        } else {
                                            // Lo está moviendo a los costados (por encima o por debajo) del nodo de destino.
                                            // Tiene que ponerlo justo encima. (no siblings)
                                            return false;
                                        }
                                    }
                                } else {
                                    return false;
                                }
                            }
                            return false;
                        },
                        'data': NewjsTreeData(config),
                        'themes': {
                            'name': 'proton',
                            'responsive': true
                        }
                    },
                    "contextmenu": {
                        items: ContextMenuItems
                    },
                    "plugins": ["dnd", "contextmenu" ,"search"]
                })
                .on('copy_node.jstree', function (e, data) {
                    // Despues de realizarse la copia de un nodo, necesito cargarle los datos asociados nuevamente
                    data.node.data = data.original.data;

                    config.Eventos.CopiandoNodo(data.original, data.node);
                });

            $1(document).on('dnd_stop.vakata', function (e, data) {
                if (data.data.jstree) {
                    config.Eventos.FinArrastre(data.element);
                }
            });
        });

    });

}
