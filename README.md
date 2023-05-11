# MongoDB NestJs Utils

[![npm version](https://img.shields.io/npm/v/mongo-nestjs-utils)](https://www.npmjs.com/package/mongo-nestjs-utils)
[![npm download by month](https://img.shields.io/npm/dm/mongo-nestjs-utils)](https://npmcharts.com/compare/mongo-nestjs-utils?minimal=true)
[![npm peer dependency version - @nestjs/core](https://img.shields.io/npm/dependency-version/mongo-nestjs-utils/peer/@nestjs/core)](https://github.com/nestjs/nest)

## Description

It's a [MongoDB](https://www.mongodb.com) database model utilities for [Nest.js](https://github.com/nestjs/nest) framework.

## Installation

```bash
$ npm install --save @hymns/mongo-nestjs-utils
```

## Quick Start

Update and extend your application entity class. Here we are using "todo" as application: `./src/todo/entities/todo.entity.ts`

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { AbstractDocument } from '@hymns/mongo-nestjs-utils';

@Schema({ versionKey: false })
export class TodoDocument extends AbstractDocument {
  @Prop()
  title: string;

  @Prop()
  isComplete: boolean;

  ...
}

export const TodoEntity = SchemaFactory.createForClass(TodoDocument);

```

Create new file for your application repository `./src/todo/todo.repository.ts`

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { AbstractRepository } from '@hymns/mongo-nestjs-utils';
import { TodoDocument } from './entities/todo.entity';

@Injectable()
export class TodoRepository extends AbstractRepository<TodoDocument> {
  protected readonly logger = new Logger(TodoRepository.name);

  constructor(
    @InjectModel(TodoDocument.name)
    todoModel: Model<TodoDocument>,
  ) {
    super(todoModel);
  }
}
```

Now you can use the repository with CRUD example on your application service.
`./src/todo/todo.service.ts`

```typescript
import { Inject, Injectable } from '@nestjs/common';
import { CreateTodoDto } from './dto/create-todo.dto';
import { UpdateTodoDto } from './dto/update-todo.dto';
import { TodoRepository } from './todo.repository';

@Injectable()
export class TodoService {
  constructor(
    private readonly todoRepository: TodoRepository,
  ) {}

  async create(createTodoDto: CreateTodoDto) {
    return this.todoRepository.create(createTodoDto);
  }

  async findAll() {
    return this.todoRepository.find({});
  }

  async findOne(_id: string) {
    return this.todoRepository.findOne({ _id });
  }

  async update(_id: string, updateTodoDto: UpdateTodoDto) {
    return this.todoRepository.findOneAndUpdate(
      { _id },
      { $set: updateTodoDto },
    );
  }

  async remove(_id: string) {
    return this.todoRepository.findOneAndDelete({ _id });
  }
}
```

Lastly register the `DatabaseModule` in your application module: `./src/todo/app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from '@hymns/mongo-nestjs-utils';
...
import { TodoService } from './todo.service';
import { TodoRepository } from './todo.repository';
import { TodoDocument, TodoEntity } from './entities/todo.entity';

@Module({
  imports: [
    DatabaseModule,
    DatabaseModule.forFeature([
      { name: TodoDocument.name, schema: TodoEntity },
    ])
    ...
  ],
  providers: [TodoService, TodoRepository],
  ...
})
export class AppModule {}
```

Make sure add the MongoDB connection URL inside your `.env` file `MONGODB_URL=mongodb://mongo:27017/todos`
