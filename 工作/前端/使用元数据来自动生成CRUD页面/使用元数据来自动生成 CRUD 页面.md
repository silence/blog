[TOC]

# 使用元数据来自动生成 CRUD 页面

在平常的基本需求迭代中有许许多多像下面这样的纯粹的 **CRUD** 页面的开发。在 B 端业务中我们不免需要写一遍又一遍相似的样板代码，虽然不同的项目业务逻辑不同，但是我们确需要写相似的 **表格展示页**、**表单**、**搜索**，这样的应用我们统称为 **CURD** 类应用。据不保守估计这样的类似应用可能会占到业务的 **30%** 以上。![image-20201206152643371](https://raw.githubusercontent.com/silence/blog/assets/assets/20201206172154.png)

---

#### low code 是什么

>A low-code development platform (LCDP) is software that provides a development environment used to create application software through graphical user interfaces and configuration instead of traditional hand-coded computer programming. A low-code model enables developers of varied experience levels to create applications using a visual user interface in combination with model-driven logic.

**lowcode 是指通过 GUI、配置化的方式代替传统的手写代码编程**，让经验背景不同的开发者都能在低代码开发平台上，基于可视化的 UI 和模型驱动的逻辑来创建应用程序。

- 利用低代码平台创建整个 App，或者只在一些特定场景需要人工编码，减少了所需的人工代码量，一方面能够提高业务交付速度，另一方面也能让广大非专业开发者参与应用开发，降低了开发门槛和人力成本
- 技术上，实现低代码平台的关键要素是**模型驱动设计、代码自动生成和可视化编程**，通过这些手段来隐藏下层的代码细节

例如阿里的[icework](https://ice.work/)**飞冰**是一个可以通过物料市场进行可视化拖拽的低代码平台。配合它们的物料市场，能极大的加速前端应用的构建。

但是我们这里不采用可视化拖拽的方式来做腾讯云控制台业务：

- 腾讯云控制台业务所依赖的组件 `tea-component` 还尚不具备拖拽生成 low-code 的能力
- 实际体验上拖拽还没复制粘贴快速，拖拽生成的代码很**差劲**，得改很多东西，还得做很多配置，结论：**不如自己写**代码
- 也考虑过通过识别设计产品原型稿prd的方式来自动识别组件类别，（ps：探索这里的时候发现有一个阿里的人工智能团队在做设计稿自动生成代码的事情，它们甚至还做了 node 跑 python 的机器学习工具包的库[pipcook](https://github.com/alibaba/pipcook)），但是这种方式需要对组件做大量的标记，这种基于 AI 的对象检测能力现在还很差，识别不准确不说，生成的 low-code 更是没法看。

#### Model-driven logic - 模型驱动设计

回到数据的 model 上来，其实我们的表单表格的核心都是基于数据来展现的，那么数据是哪来的呢？数据库里存取的，这些数据库里存的数据都可以称之为**元数据**，这里我们接小刀分享的文章[前端元编程——使用注解加速你的前端开发](https://zhuanlan.zhihu.com/p/274328551)介绍的前端元编程的概念，我们使用 typescript 里的 `Decorator`，`Reflect Metadata ` 来定义和提取元数据。 
- 复习一下，我们的核心是要**数据驱动前端开发**

![image-20201206161706246](https://raw.githubusercontent.com/silence/blog/assets/assets/20201206172213.png)

- 借助 `Decorator` 和 `Reflect` 将 CRUD 页面所需的样板类方法属性元编程在 Model 上。

```typescript
export class Person {
  @TableColumn({ header: "ID" })
  id: string;

  @FormField({
    label: "名字",
    validationSchema: Yup.string().max(20, "姓名最多20个字符").required("名字不能为空"),
  })
  @TableColumn({ header: "名字" })
  name: string;

  @FormField({
    label: "年龄",
    validationSchema: Yup.number().required("年龄不能为空"),
  })
  @TableColumn({ header: "年龄" })
  age: number;

  @FormField({
    label: "性别",
    validationSchema: Yup.string().required("性别不能为空"),
    enum: Gender,
  })
  @TableColumn({ header: "性别" })
  gender: Gender;

  @FormField({
    label: "爱好",
    validationSchema: Yup.array().required("爱好不能为空"),
    enum: Hobbies,
  })
  @TableColumn({
    header: "爱好",
    render: (record: Person) => record.hobbies.join(","),
  })
  hobbies: Hobbies[];

  @FormField({
    label: "是否需要996",
    validationSchema: Yup.boolean().required("不能为空"),
  })
  @TableColumn({
    header: "是否需要996",
    render: (record: Person) => (record.is996 ? "是" : "否"),
  })
  is996: boolean;
}
```

相应的 `Decorator` 的定义如下：

```typescript
// 表格Column元数据和表格 propertyKey list
export const COLUMN_METADATA = "column";
export const COLUMN_LIST = [] as string[];

// 表单field元数据和表单 propertyKey list
export const FORM_FIELD_METADATA = "form";
export const FORM_FIELD_LIST = [] as string[];

export interface IColumnOptions {
  /** 表格头 */
  header: string;
  /** 表格渲染方法 */
  render?: (record) => React.ReactNode;
}

export function Column(options: IColumnOptions): PropertyDecorator {
  return (target, propertyKey: string) => {
    Reflect.defineMetadata(COLUMN_METADATA, options, target, propertyKey);
    COLUMN_LIST.push(propertyKey);
  };
}

export interface IFormFieldOptions {
  /** 表单label */
  label: string;
  /** 表单验证属性 */
  validationSchema?: any;
  /** 枚举属性 */
  enum?: Object | (string | number)[];
}

export function FormField(options: IFormFieldOptions): PropertyDecorator {
  return (target, propertyKey: string) => {
    Reflect.defineMetadata(FORM_FIELD_METADATA, options, target, propertyKey);
    FORM_FIELD_LIST.push(propertyKey);
  };
}
```

- 但这里我的实现和 tyler 的实现有些许所差别，tyler 是定义了一个 EnhancedClass 的 Decorator 来给数据 Model 类注入获取 table 和 form colomn 和 config 的方法。我的实现上并没有使用一个ClassDecorator。而是传入这个 class 直接通过元数据去获取的。
- 这里我的 Person Class 里的属性值并没有赋值，即这个类实例化后将是一个空对象，该类实例化**没有任何作用**，所有的属性都是通过元数据去获取的。
- 在这里我通过 ts 的强类型的性质，根据类的属性值的类型来定义表单控件，如 `String` 就对应 ` Input` 组件 ,  `Number` 就对应 ` Input type=number` 的组件，`Array` 就对应 `Checkbox` 组件，`Boolean` 就对应`Switch` 组件。如果 `String` 再定义 `enum` 属性就会变成 `Radio` 组件。

```typescript
/** 定义一个从元数据里获取FormField的函数，Type<any>代表类的类型 */
export function getFormField<T = any>(target: Type<T>) {
  /** FORM_FIELD_LIST存放的是表格FormField元数据的列表，通过Reflect.getMetadata来获取类里给元数据定义的类型和选项 */
  const fields = FORM_FIELD_LIST.map((propertyKey) => {
    const type: Function = Reflect.getMetadata("design:type", target.prototype, propertyKey);
    const options: IFormFieldOptions = Reflect.getMetadata(FORM_FIELD_METADATA, target.prototype, propertyKey);
    const fieldOptions = {
      name: propertyKey,
      label: options.label,
    };
    switch (type.name) {
      case "String":
        return options.enum ? (
        /** 若Form选项里有enum选项则定义为Radio组件，否则定义为Input组件 */
          <RadioField
            {...fieldOptions}
            options={Object.values(options.enum).map((el) => ({
              value: el,
              text: el,
            }))}
            key={propertyKey}
          />
        ) : (
          <InputField {...fieldOptions} size="full" key={propertyKey} />
        );
      case "Number":
        /** 给Input组件定义type="number"属性 */
        return <InputField {...fieldOptions} type="number" size="full" key={propertyKey} />;
      case "Array":
        /** 若为Array类型，则定义为Checkbox组件 */
        return (
          <CheckboxField
            {...fieldOptions}
            options={Object.values(options.enum).map((el) => ({
              value: el,
              text: el,
            }))}
            key={propertyKey}
          />
        );
      /** 若为Boolean类型，则定义为Switch组件 */
      case "Boolean":
        return <SwitchField {...fieldOptions} key={propertyKey} />;
    }
  });
  return fields;
}
```

- 这样我们就可以从 `Person Class` 直接生成所需要的**列表**和**表单**展示页了。但是我们还缺少最重要的 api 请求接口。

到目前为止都只是前端的 Model 生成 Form 和 Table 的内容，但是其实数据库也是可以根据元数据生成的。

#### 利用 ORM 来通过元数据生成后端 CRUD 接口

![image-20201206161941145](https://raw.githubusercontent.com/silence/blog/assets/assets/20201206172230.png)

ORM全称是：Object Relational Mapping(对象关系映射)，其主要作用是在编程中，把面向对象的概念跟数据库中表的概念对应起来。举例来说就是，我定义一个对象，那就对应着一张表，这个对象的实例，就对应着表中的一条记录。

- 我们可以通过 **ORM** 将同一个 Model Class 转换成数据库里的一个表，再通过现有 restful 接口规范，我们甚至可以自动生成 restful 规范的接口，包括增删查改、甚至有条件搜索等。
- 我选取的是 TypeOrm 这个框架，这个框架是纯 ts 开发的，使用 Decorator 的语法来定义 entity，即天生就支持元数据。

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';
import { ApiProperty } from '@nestjs/swagger';

type Gender = 'male' | 'female';
type Hobbies =
  | 'basketball'
  | 'badminton'
  | 'billiards'
  | 'football'
  | 'movie'
  | 'hiking';

@Entity('persons')
export class PersonEntity {
  @PrimaryGeneratedColumn()
  id: string;

  @ApiProperty()
  @Column()
  name: string;

  @ApiProperty()
  @Column()
  age: number;

  @ApiProperty()
  @Column({
    type: 'enum',
    enum: ['male', 'female'],
    default: 'male',
  })
  gender: Gender;

  @ApiProperty()
  @Column({
    type: 'set',
    enum: [
      'basketball',
      'badminton',
      'billiards',
      'football',
      'movie',
      'hiking',
    ],
    default: [],
  })
  hobbies: Hobbies[];

  @ApiProperty()
  @Column()
  is996: boolean;

  @CreateDateColumn()
  create_at: Date;

  @UpdateDateColumn()
  update_at: Date;
}
```

- 再使用了 nestjsx/crud 这个框架后自动生成 restful 规范的接口，效果如展示

```typescript
import { Controller } from '@nestjs/common';
import { Crud, CrudController } from '@nestjsx/crud';
import { PersonEntity } from './person.entity';
import { PersonService } from './person.service';

@Crud({
  model: {
    type: PersonEntity,
  },
})
@Controller('person')
export class PersonController implements CrudController<PersonEntity> {
  constructor(public service: PersonService) {}
}
```

![image-20201206171801115](https://raw.githubusercontent.com/silence/blog/assets/assets/20201206172246.png)

这样我们就实现了一个由**元数据**驱动的CRUD前端应用。

#### 总结

1. 这种前后端独立应用开发的开发模式正好对应微前端+后台微服务化的开发模式。

2. 最近 BFF 的概念很火，可以让后端将 ui 对应的接口我们自己来维护，即将后端的 Controller 层接过来，这样就不会有前后端沟通某个数据排序谁来做的问题了（ps：不过这样的话前端就得干更多的活了😓，让后端更专注于底层）
3. 这样做的缺点也是显而易见的，像这样将部分业务逻辑耦合到 Model 层后，若后续需要添加更多的功能，Model 层会越来越庞大，也越来越难以维护，所以我建议只针对这种纯粹的 CRUD 应用采用这种前端元编程方式，不需要过多的配置，由数据 Model 直出页面布局。但是更复杂的前端应用就不合适了。当然这里也只是提供一种思路。相信未来大家一定能有更好的办法来解决现在的问题的~