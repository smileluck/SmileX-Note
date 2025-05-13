[toc]

---

# nestjs-项目搭建
## 初始化

### 1. 安装nest-cli
```shell
npm i -g @nestjs/cli
```

### 2. 创建项目
```shell
nest new project-name

# 要创建具有更严格功能集的新 TypeScript 项目，请将 --strict 标志传递给 nest new 命令。
```

### 3. 运行项目

```shell
cd project-name
npm run start
```

## 创建模块
### 1. .env文件配置功能
> 官方中文：https://nest.nodejs.cn/techniques/configuration

1. 安装依赖
```shell
npm i --save @nestjs/config
```
2. 创建.env文件
```
serve.port=3000

# 数据库配置
## mysql
database.enable=true
  ## 主数据库
database.main.enable=true
database.main.type='mysql'
database.main.url="mysql://root:123456@localhost:3306/smilex-db?charset=utf8mb4"
# database.main.host='localhost'
# database.main.port=3306
# database.main.username='root'
# database.main.password='123456'
# database.main.database='smilex-db'
# database.main.charset='utf8mb4'

# redis配置
redis.enable=false
redis.type='single'
redis.host='127.0.0.1'
redis.port=6379
redis.password=''
redis.db=0
```
3. 创建 src/config/config.module.ts
```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { loadConfigEnv } from './configuration';
import { validate } from './config.validate';
import { ConfigModule } from '@nestjs/config';

@Module({})
export class ConfigurationModule {
    static forRoot(): DynamicModule {
        return {
            module: ConfigurationModule,
            imports: [
                ConfigModule.forRoot({
                    isGlobal: true,
                    validate,
                    envFilePath: loadConfigEnv()
                    // load: [loadConfig]
                }),],
        };
    }
}
```
4. 创建 src/config/configuration.ts
```typescript
// 配置文件
import { readFileSync } from 'fs';
import * as yaml from 'js-yaml';
import { join } from 'path';
import { getEnumKeyByValue } from "../utils/enums.helper"
import { NodeEnvFileEnum } from './config.constant';

// 读取配置文件
const loadConfigFile = (node_env: NodeEnvFileEnum) => {
    try {
        const parentDir = process.cwd()
        return yaml.load(
            readFileSync(join(parentDir, node_env), 'utf8'),
        ) as Record<string, any>;
    } catch (e) {
        console.error(e);
        process.exit(1);
    }
}

// 读取配置
const loadConfig = () => {
    try {
        // 检查配置文件
        const config = loadConfigFile(NodeEnvFileEnum.default)
        // 如果 config 中的Node_Env存在，就读取对应文件并覆盖config
        if (process.env.NODE_ENV) {
            const node_env = NodeEnvFileEnum[config.NODE_ENV];
            const node_env_config = loadConfigFile(node_env);
            Object.assign(config, node_env_config);
        }
        return config;
    } catch (error) {
        console.log(error);
        process.exit(1);
    }
};

// 读取配置文件路径
const loadConfigEnv = () => {
    const envs: string[] = []
    envs.push(NodeEnvFileEnum.default)
    // const e = getEnumKeyByValue(NodeEnvFileEnum, process.env.NODE_ENV as any)
    // if (e) {
    //     envs.push(e)
    // }
    return envs
}

export {
    loadConfig,
    loadConfigEnv
}
```
5. 创建 src/config/config.constant.ts
```typescript

// 环境枚举
enum NodeEnvEnum {
    Development = "dev",
    Production = "prod",
    Test = "test",
}


enum NodeEnvFileEnum {
    default = ".env",
    Development = ".env.dev",
    Test = ".env.test",
    Production = ".env.prod",
}

// 数据库方言枚举
enum DBDialectEnum {
    Mysql = "mysql",
    Mongo = "mongo",
    Postgres = "postgres"
    // ...
}



// Redis类型枚举
enum RedisTypeEnum {
    Single = "single",
    Cluster = "cluster",
    Sentinel = "sentinel"
}

export {
    NodeEnvEnum,
    NodeEnvFileEnum,
    DBDialectEnum,
    RedisTypeEnum
}
```
6. 创建 src/config/config.validate.ts

```typescript
// 环境验证
import { plainToInstance } from 'class-transformer';
import { validateSync } from 'class-validator';
import { EnvVars } from './config.type';

export function validate<T extends object>(config: Record<string, unknown>) {
    const validatedConfig = plainToInstance(
        EnvVars,
        config,
        { enableImplicitConversion: true },
    );
    const errors = validateSync(validatedConfig, { skipMissingProperties: false });

    if (errors.length > 0) {
        throw new Error(errors.toString());
    }
    return validatedConfig;
}

```

7. 创建 src/config/config.type.ts

```typescript
import { IsEnum, IsNumber, Max, Min, IsBoolean, IsString, IsNotEmpty, IsObject, IsOptional, IsInstance, ValidateIf, IsArray } from 'class-validator';
import { DBDialectEnum, NodeEnvEnum, RedisTypeEnum } from './config.constant';

/**
 * TypeORM 配置
 */
class EnvDataBaseConnVars {

    @IsOptional()
    @IsBoolean()
    enable: boolean = false


    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    name: string

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsEnum(DBDialectEnum)
    type: DBDialectEnum

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    host: string

    @ValidateIf((obj) => obj.enable)
    @IsNumber()
    @Min(0)
    @Max(65535)
    port: number;

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    username: string;

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    password: string;

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    database: string;

    @ValidateIf((obj) => obj.enable)
    @IsOptional()
    @IsString()
    charset: string = "utf8mb4";


    // 相对路径
    @ValidateIf((obj) => obj.enable)
    // @IsOptional()
    @IsArray()
    entities: string[] = ['/**/*.entity{.ts,.js}']
}

class EnvPrismaConnVars {
    @IsOptional()
    @IsBoolean()
    enable: boolean = false

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsEnum(DBDialectEnum)
    type: DBDialectEnum

    @ValidateIf((obj) => obj.enable)
    @IsString()
    url: string;
}

// 数据库配置模型
class EnvDataBaseVars {
    @IsBoolean()
    enable: boolean

    // 主数据库, 不能为空
    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    main: EnvPrismaConnVars

    // @ValidateIf((obj) => obj.enable)
    // slave: EnvPrismaConnVars[]
}
// redis
class EnvRedisVars {

    @IsBoolean()
    enable: boolean = false

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsEnum(RedisTypeEnum)
    type: RedisTypeEnum

    @ValidateIf((obj) => obj.enable)
    @IsNotEmpty()
    @IsString()
    host: string

    @ValidateIf((obj) => obj.enable)
    @IsNumber()
    @Min(0)
    @Max(65535)
    port: number;

    @ValidateIf((obj) => obj.enable)
    @IsOptional()
    @IsString()
    password: string = '';

    @ValidateIf((obj) => obj.enable)

    @IsNumber()
    @Min(0)
    @Max(65535)
    db: number;
}


class EnvVars {
    @IsOptional()
    @IsEnum(NodeEnvEnum)
    node_env: NodeEnvEnum;

    @IsOptional()
    @IsString()
    ip: string = '0.0.0.0';

    @IsNumber()
    @Min(0)
    @Max(65535)
    port: number = 3000;

    @IsOptional()
    @IsInstance(EnvDataBaseVars)
    database: EnvDataBaseVars;

    @IsOptional()
    @IsInstance(EnvRedisVars)
    redis: EnvRedisVars
}

export {
    EnvDataBaseVars,
    EnvDataBaseConnVars,
    EnvVars,
    NodeEnvEnum,
}
```
