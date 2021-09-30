***projeto gerenciador de tarefas***
1)chegar na pasta do projeto pelo terminal (projetoAngularCalculadora)
> ng new gerenciador-de-tarefas

2)entrar na pasta gerenciador-de-tarefas:
> cd gerenciador-de-tarefas
> ng serve

3)Remover dois últimos blocos de app.component.spec.ts
4)em app.component.ts
title = 'Gerenciador de Tarefas';
5)limpar app.component.html
<div class="container-fluid">
  <nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="nav-header">
        <a href="" class="navbar-brand">
          {{ title }}
        </a>
      </div>
    </div>
  </nav>
</div>

6)novo terminal entrar na pasta que criamos dentro do projeto:
C:\Users\diana\Desktop\Angular_aulas\projetoAngularCalculadora\gerenciador-de-tarefas>npm install --save bootstrap@3

7)indicar o caminho do bootstrap em angular.json:
passar o caminho do bootstrap no style: 
"styles": [
              "src/styles.css",
              "node_modules/bootstrap/dist/css/bootstrap.min.css"
            ],
8)reiniciar o servidor:
>ng serve
9)Utilizar rotas para permitir a troca de componentes da página - existe um módulo exclusivo pra rotas
é necessário configurar as rotas no conteúdo principal

em src>app> criar um arquivo app-routing.module.ts: ou (ng g m [ModuleName] --routing)
e fazer importações:
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

export const router : Routes = [];

@NgModule({
    imports :[ RouterModule.forRoot(router)],
    exports :[ RouterModule]
})

export class AppRoutingModule {}


10)informar ao app.module.ts da criação desse novo módulo:
imports: [
    BrowserModule, 
    AppRoutingModule //aqui informamos os modulos criados
  ],

11)em app.component.html:
<!-- abaixo da barra de navegação  -->
  <router-outlet></router-outlet> 

12)criar novo módulo, o de tarefas:
> ng g module tarefas
13)criar index.ts dentro de tarefas:
export * from './tarefas.module'
14) em app.module.ts
imports: [
    BrowserModule, 
    AppRoutingModule, //aqui informamos os modulos criados
    TarefasModule
  ],
15)criar um local onde possa fazer a criação de tarefas:
criar model - moldar o nosso objeto
dentro da pasta tarefas criar pasta com o nome shared (aqui dentro ficaram os arquivos que queremos compartilhar)
então dentro dessa pasta vamos criar um arquivo chamado:
** tarefa.model.ts que recebe:

export class Tarefa{
    constructor(
        public id?:number,
        public nome?:string,
        public concluida?:boolean
    ) {

    }
}

ir em ts.config.json trocar a linha 8 para: "strict": false,

16) criar um index.ts dentro de shared:
export * from './tarefa.model';

17)dentro do index.ts sentro do módulo "O que foi criado, neste caso: livros" incluir o caminho da pasta shared:
export * from './shared';

*nesse arquivo teremos então: 
export * from './tarefas.module';
export * from './shared';

18)criar um serviço com nome tarefa dentro de livros/shared
C:\Users\diana\Desktop\Angular_aulas\projetoAngularCalculadora\gerenciador-de-tarefas>ng g service atendimento/shared/atendimento

no index.ts da pasta shared:
export * from './tarefa.model';
export * from './tarefa.service';


19)precisamos informar ao tarefas.module.ts a existencia desse serviço criado(para estar disponível ao app.module)
providers:[
    TarefaService
  ]

O arquivo tarefas.module.ts devemos fica com a seguinte composição:
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TarefaService } from './shared';



@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  providers:[
    TarefaService
  ]
})
export class TarefasModule { }

20)Agora podemos implementar o serviço - onde ficará a parte lógica da aplicação (em **tarefa.service.ts**)
 precisaremos dos métodos que serão implementados por métodos:
- listar
- cadastrar
- buscar por id
- atualizar
- remover
- alterar status

excluímos: a seguinte parte do código para não fazer a injeção na raiz:
{
  providedIn: 'root'
}

21)Vamos iniciar a criação dos métodos, que funcionaram como uma função
1- método listarTodos: 
import { Injectable } from '@angular/core';
import { Tarefa } from './tarefa.model'

@Injectable()
export class TarefaService {

  constructor() { }

  listarTodos(): Tarefa[]{  //esse Tarefa vem do model

  }
}

atualizando a criação desse serviço:
import { Injectable } from '@angular/core';
import { Tarefa } from './tarefa.model'

@Injectable()
export class TarefaService {

  constructor() { }

  listarTodos(): Tarefa[]{ //vem de model
    const tarefas = localStorage['tarefas'] //vai receber do localStorage['nomedobanco']
    return tarefas ? JSON.parse(tarefas): []; //dentro da const tarefas recebe todas as tarefas, mas no primeiro momento não terá nenhuma tarefa, então dentro do return vamos fazer a verificação. JSON.parse converterá de string para json (localStorage armazena como string)
  }
}


2- método cadastrar
cadastrar(tarefa: Tarefa):void{  //passar o parametro do que quero cadastrar
    const tarefas = this.listarTodos();
    tarefa.id = new Date().getTime();  //usamos o método timestamp como id por ser um identificador único
    tarefas.push(tarefa);
    localStorage['tarefas'] = JSON.stringify(tarefas)
  }

arquivo deve ficar assim:
import { Injectable } from '@angular/core';
import { Tarefa } from './tarefa.model'

@Injectable()
export class TarefaService {

  constructor() { }

  listarTodos(): Tarefa[]{ //vem de model
    const tarefas = localStorage['tarefas'] 
    return tarefas ? JSON.parse(tarefas): []; 
  }

  cadastrar(tarefa: Tarefa):void{  
    const tarefas = this.listarTodos();
    tarefa.id = new Date().getTime();
    tarefas.push(tarefa);
    localStorage['tarefas'] = JSON.stringify(tarefas)
  }

  buscarPorId(id: number):Tarefa{
    const tarefas : Tarefa[] = this.listarTodos();
    return tarefas.find(tarefa => tarefa.id == id );
  }

  atualizar(tarefa: Tarefa): void{
    const tarefas : Tarefa[] = this.listarTodos();
    tarefas.forEach((obj, index, objs) =>{
      if( tarefa.id === obj.id){
        objs[index] = tarefa;
      }
    });
    localStorage['tarefas'] = JSON.stringify(tarefas); 
  }

  remover(id: number):void{
    let tarefas: Tarefa[] = this.listarTodos();
    tarefas = tarefas.filter(tarefas => tarefas.id !== id);
    localStorage['tarefas'] = JSON.stringify(tarefas);
  }

  alterarStatus(id: number):void{
    const tarefas : Tarefa[] = this.listarTodos();
    tarefas.forEach((obj, index, objs)=>{
      if(id === obj.id){
        objs[index].concluida = !obj.concluida;
      }
    });
    localStorage['tarefas'] = JSON.stringify(tarefas);
  }

}


20) Criar o component de listar-tarefas:
C:\Users\diana\Desktop\Angular_aulas\projetoAngularCalculadora\gerenciador-de-tarefas>ng g component atendimento/listar-atendimentos
renomear a pasta criada para listar
21)criar barrels nessa pasta = index.ts:
export * from './listar-tarefa.component';

22)no index.ts da pasta shared incluir o export da pasta listar:
export * from '../listar';

23)Confirmar em tarefas.module.ts se ocorreu a importação de import { ListarTarefaComponent } from './listar';

declarations: [
    ListarTarefaComponent
  ],

24)Agora vamos precisar criar rota pra esse componente
dentro de tarefas criar o arquivo tarefas-routing.module.ts
import { Routes } from '@angular/router';
import { ListarTarefaComponent } from './listar';

export const TarefaRoutes: Routes = [
    { 
        path:'tarefas',
        redirectTo: 'tarefas/listar'
    }, 
    { 
        path:'tarefas/listar',
        component: ListarTarefaComponent
    }
];

25)em app-routing.module.ts incluir import { NgModule } from '@angular/core';
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { TarefaRoutes } from './tarefas';

export const router : Routes = [
    {
        path: '',
        redirectTo: 'tarefas/listar',
        pathMatch:'full'  //redireciona direto pra pasta raiz
    }, 
    ...TarefaRoutes  //faz um merge das rotas de tarefas-routing.module aqui
]; //dentro dos colchotes irão nossas rotas. faz concatenação de um array de rotas

@NgModule({
    imports : [ RouterModule.forRoot(router)],
    exports : [ 
        RouterModule
    ]
})

export class AppRoutingModule {} //informa sua existencia em app.module.ts

26)no listar-tarefa.component.html 
vai o html que queremos renderizar

27)em tarefas.module.ts:
importar 

 imports: [
    CommonModule,
    RouterModule,
    FormsModule
  ],

import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';

28)em listar-tarefa.component.ts vamos chamar os métodos que criamos em serviços por data binding:
import { Component, OnInit } from '@angular/core';

import { TarefaService, Tarefa} from '../shared';


@Component({
  selector: 'app-listar-tarefa',
  templateUrl: './listar-tarefa.component.html',
  styleUrls: ['./listar-tarefa.component.css']
})
export class ListarTarefaComponent implements OnInit {

  tarefas: Tarefa[];  //vem do model

  constructor(private tarefaService : TarefaService) { }

  ngOnInit() {
    this.tarefas = this.listarTodos();
  }
  listarTodos(): Tarefa[]{
    return this.tarefaService.listarTodos();
  }

}

29)No listar-tarefa component.html 

<tr *ngFor="let tarefa of tarefas" [class.warning]="!tarefa.concluida">
            <td >
                {{ tarefa.nome}}
                <!-- [tarefaConcluida] ="tarefa.concluida" -->
            </td>

<p *ngIf="tarefas.length==0">Nenhuma tarefa cadastrada.</p>


///////////////////////////////////////////////////////////

Cadastrar tarefas

1 - Criar novo componente:

ng g component atendimento/cadastrar-atendimento

2 -renomear a nova pasta (componente) cadastrar-tarefa para cadastrar

barril index.ts

incluir no index de tarefas cadastrar

2 - incluir no index.ts de tarefas: 

export * from './cadastrar';


3 - Em tarefas-routin-module.ts: incluir CadastrarTarefaComponent, conforme abaixo:

import { Routes } from "@angular/router";
import { CadastrarTarefaComponent } from "./cadastrar";
import { ListarTarefaComponent } from "./listar";


export const TarefaRoutes : Routes = [
    {
        path:'tarefas',
        redirectTo: 'tarefas/listar'
    },
    {
        path: 'tarefas/listar',
        component: ListarTarefaComponent
    },
    {
        path:'tarefas/cadastrar',
        component: CadastrarTarefaComponent
    }
];

4 - No listar-tarefa-component.html

Editar o HTML Data-bind do tipo propriety [routerLink]="['/tarefas/cadastrar']"

assim:

...
<th class="text-center">
                <a class="btn btn-xs btn-success" [routerLink]="['/tarefas/cadastrar']">
                    <span class="glyphicon glyphicon-plus" aria-hidden="true"></span> Novo
                </a>
            </th>

...

5 - Arquivo HTML modelo da Sayure em cadastrar-tarefa-component.html

<h1>Cadastrar tarefa</h1>

<div class="well">
  <form #formTarefa="ngForm">
    <div class="form-group">
      <label for="nome">Tarefa:</label>
      <input type="text" class="form-control" id="nome" name="nome" minlength="5" required> 
      <div class="alert alert-danger">
        <div>
          Digite a tarefa.
        </div>
        <div>
          A tarefa deve conter ao menos 5 caracteres.
        </div>
      </div>
    </div>
    <div class="form-group text-center">
      <input type="submit" class="btn btn-success">
      <a class="btn btn-default">
        <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span> Voltar
      </a>
    </div>
  </form>
</div>

6 - em cadastrar-tarefa-component.ts import NgForm e Router, bem como criar o decorador e o construtor.

import { Component, OnInit, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';
import { Router } from '@angular/router';

import { TarefaService, Tarefa } from '../shared';

@Component({
  selector: 'app-cadastrar-tarefa',
  templateUrl: './cadastrar-tarefa.component.html',
  styleUrls: ['./cadastrar-tarefa.component.css']
})
export class CadastrarTarefaComponent implements OnInit {

  @ViewChild('formTarefa', {static:true})formTarefa: NgForm;

  tarefa:Tarefa;


  constructor(private tarefaService: TarefaService, private router:Router) { }

  ngOnInit() {
    this.tarefa = new Tarefa();
  }

  cadastrar(): void {
    if (this.formTarefa.form.valid) {
      this.tarefaService.cadastrar(this.tarefa);
      this.router.navigate(['/tarefas']);
    }

  }

}

7- Conectar o componente com o imput do html: two way bind [(ngModel)]="tarefa.nome" and more:
html:

<h1>Cadastrar tarefa</h1>

<div class="well">
    <form #formTarefa="ngForm">
        <div class="form-group">
            <label for="nome">Tarefa:</label>
            <input type="text" class="form-control" id="nome" name="nome" [(ngModel)]="tarefa.nome" #nome="ngModel"
                minlength="5" required>
            <div *ngIf="nome.errors && (nome.dirty || nome.touched)" class="alert alert-danger">
                <div [hidden]="!nome.errors.required">
                    Digite a tarefa.
                </div>
                <div [hidden]="!nome.errors.minlength">
                    A tarefa deve conter ao menos 5 caracteres.
                </div>
            </div>
        </div>
        <div class="form-group text-center">
            <input type="submit" class="btn btn-success" (click)="cadastrar()" value="Cadastrar" [disabled]="!formTarefa.form.valid">
            <a class="btn btn-default" [routerLink]="['/tarefas']">
                <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span> Voltar
            </a>
        </div>
    </form>
</div>


//////////////////////////////////////

Alterar status

1 - listar-tarefa-component.ts

alterarStatus(tarefa:Tarefa): void {
    if (confirm(`Deseja alterar o status da tarefa ${tarefa.nome} ?`)) {
      this.tarefaService.alterartatus(tarefa.id);
      this.tarefas = this.listarTodos()
    }
  }

2 - listar-tarefa-component.html

<td style="width: 70px" class="text-center">
                <input type="checkbox" [value]="tarefa.id" [checked]="tarefa.concluida" (click)="alterarStatus(tarefa)">
            </td>

//////////////////

remover

1 - em listar-tarefa-component.ts

  remover($event:any, tarefa:Tarefa):void{
    $event.preventDefault();
    if (confirm(`Deseja remover a tarefa ${tarefa.nome} ?`)) {
      this.tarefaService.remover(tarefa.id);
      this.tarefas = this.listarTodos()
    }
  }

2 - em listar-tarefa-component.html incluir (click)="remover($event, tarefa)"

<a href="#" title="Remover" alt="Remover" class="btn btn-xs btn-danger" (click)="remover($event, tarefa)">
                    <span class="glyphicon glyphicon-remove" aria-hidden="true"></span> Remover
                </a>


////////
editar

1 -ng g c atendimento/editar-atendimento

2 - renomeia a pasta editar-tarefas para editar

3 - no index do tarefas:

export * from './editar'

4- em editar.module renomeia o import

import { EditarTarefaComponent } from './editar';

5 - tarefas routin module:

{
        path: 'tarefas/editar/:id',
        component: EditarTarefaComponent
    }
renomear o import para import { EditarTarefaComponent } from "./editar";

6 - Listar component html => [routerLink]="['/tarefas/editar', tarefa.id]"

<a title="Editar" alt="Editar" class="btn btn-xs btn-info" [routerLink]="['/tarefas/editar', tarefa.id]">
                    <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span> Editar
                </a>


7 -em editar tarefa component

import { Component, OnInit, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';
import { TarefaService, Tarefa } from '../shared';
import { Router, ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-editar-tarefa',
  templateUrl: './editar-tarefa.component.html',
  styleUrls: ['./editar-tarefa.component.css']
})
export class EditarTarefaComponent implements OnInit {

  @ViewChild('formTarefa', {static:true})formTarefa: NgForm;

  tarefa:Tarefa;

  constructor(
    private tarefaService: TarefaService,
    private route: ActivatedRoute,
    private router: Router
    ) { }

  ngOnInit(): void {
    const id = this.route.snapshot.params['id'];
    this.tarefa = this.tarefaService.buscarPorId(id);
  }

  atualizar():void{
    if (this.formTarefa.form.valid) {
      this.tarefaService.atualizar(this.tarefa);
      this.router.navigate(['/tarefas']);
    }
  }

}

9 - em editar tarefa html

<h1>Editar tarefa</h1>

<div class="well">
    <form #formTarefa="ngForm">
        <div class="form-group">
            <label for="nome">Tarefa:</label>
            <input type="text" class="form-control" id="nome" name="nome" [(ngModel)]="tarefa.nome" #nome="ngModel"
                minlength="5" required>
            <div *ngIf="nome.errors && (nome.dirty || nome.touched)" class="alert alert-danger">
                <div [hidden]="!nome.errors.required">
                    Digite a tarefa.
                </div>
                <div [hidden]="!nome.errors.minlength">
                    A tarefa deve conter ao menos 5 caracteres.
                </div>
            </div>
        </div>
        <div class="form-group text-center">
            <input type="submit" class="btn btn-success" (click)="atualizar()" value="Atualizar" [disabled]="!formTarefa.form.valid">
            <a class="btn btn-default" [routerLink]="['/tarefas']">
                <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span> Voltar
            </a>
        </div>
    </form>
</div>

////
Tips

{{}} - String de interpolação
[   ] - Data bind - Propety

[(   )] - two way databind => ida e volta

(  ) - databind de evento Ex: Click

*ngIf - Diretiva condicional
*ngFor


/////////////////////////////////

CRIANDO DIRETIVAS

1 - ng g directive atendimento/shared/atendimento-concluido

cria tarefa-concluida-directive
barril

2 - tarefa module ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TarefaService, TarefaConcluidaDirective } from './shared';
import { ListarTarefaComponent } from './listar';
import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { CadastrarTarefaComponent } from './cadastrar';
import { EditarTarefaComponent } from './editar/editar-tarefa.component';




@NgModule({
  declarations: [
    ListarTarefaComponent,
    CadastrarTarefaComponent,
    EditarTarefaComponent,
    TarefaConcluidaDirective
  ],
  imports: [
    CommonModule,
    RouterModule,
    FormsModule
  ],
  providers:[
    TarefaService
  ]
})
export class TarefasModule { }



3 em tarefa concluida diretiva:

import { Directive, ElementRef, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[tarefaConcluida]'
})
export class TarefaConcluidaDirective implements OnInit{

  @Input() tarefaConcluida: boolean;

  constructor(private el:ElementRef) { }

  ngOnInit(){
    if (this.tarefaConcluida) {
      this.el.nativeElement.style.textDecoration = 'line-through';
    }
  }
}

4 - no listar html inclui:  [tarefaConcluida]="tarefa.concluida" 

<h1>Tarefas</h1>

<table class="table table-striped table-bordered table-hover">
    <thead>
        <tr>
            <th>Tarefa</th>
            <th>Concluída</th>
            <th class="text-center">
                <a class="btn btn-xs btn-success" [routerLink]="['/tarefas/cadastrar']">
                    <span class="glyphicon glyphicon-plus" aria-hidden="true"></span> Novo
                </a>
            </th>
        </tr>
    </thead>
    <tbody>
        <tr *ngFor="let tarefa of tarefas">
            <td [tarefaConcluida]="tarefa.concluida" [class.success]="tarefa.concluida" >
                {{ tarefa.nome }}
            </td>
            <td style="width: 70px" class="text-center">
                <input type="checkbox" [value]="tarefa.id" [checked]="tarefa.concluida" (click)="alterarStatus(tarefa)">
            </td>
            <td class="text-center" style="width: 200px">
                <a title="Editar" alt="Editar" class="btn btn-xs btn-info" [routerLink]="['/tarefas/editar', tarefa.id]">
                    <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span> Editar
                </a>
                <a href="#" title="Remover" alt="Remover" class="btn btn-xs btn-danger" (click)="remover($event, tarefa)">
                    <span class="glyphicon glyphicon-remove" aria-hidden="true"></span> Remover
                </a>
            </td>
        </tr>
    </tbody>
</table>

<p *ngIf="tarefas.length == 0">Nenhuma tarefa cadastrada.</p>


















