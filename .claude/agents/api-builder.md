# API Builder Agent

Invocable agent during development sessions to implement NestJS API modules with full CRUD functionality, validation, and business logic.

## Prerequisites

- NestJS project initialized
- TypeORM configured with PostgreSQL
- Database connection configured (see Database Setup below)

## Database Setup (Supabase MVP)

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        url: configService.get('DATABASE_URL'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,  // Usar migrations
        ssl: configService.get('NODE_ENV') === 'production'
          ? { rejectUnauthorized: false }  // Requerido para Supabase
          : false,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

```bash
# .env (MVP con Supabase)
DATABASE_URL=postgresql://postgres.[ref]:[pass]@aws-0-[region].pooler.supabase.com:6543/postgres
NODE_ENV=development

# .env (Producción con AWS RDS)
DATABASE_URL=postgresql://user:pass@your-rds.region.rds.amazonaws.com:5432/dbname
NODE_ENV=production
```

## Invocation

From a session, invoke with Task tool:

```
Task: API Builder Agent
Prompt: |
  Module: [module name, e.g., users, orders, products]
  Endpoints:
    - GET /api/v1/[resource] - [description]
    - POST /api/v1/[resource] - [description]
    - GET /api/v1/[resource]/:id - [description]
    - PATCH /api/v1/[resource]/:id - [description]
    - DELETE /api/v1/[resource]/:id - [description]
  Entity: |
    [TypeORM entity fields and relationships]
  Business Logic: |
    - [validation rules]
    - [business rules]
    - [side effects: emails, notifications, etc.]
  Context: [any relevant projectSpec context]
```

---

## Agent Workflow

### Step 1: Understand Module Requirements

Parse the invocation to extract:
- Module name (singular and plural forms)
- Required endpoints and their functionality
- Entity fields and relationships
- Business logic rules
- Side effects (emails, notifications, queue jobs)

### Step 2: Check Existing Code

Scan the project structure:

```bash
# Check existing modules
ls src/modules/

# Check common utilities
ls src/common/

# Check existing entities
ls src/database/entities/
```

Identify:
- Related modules to import
- Common utilities to reuse
- Existing patterns to follow

### Step 3: Generate Module Structure

Create the NestJS module with this structure:

```
src/modules/[feature]/
├── [feature].module.ts
├── [feature].controller.ts
├── [feature].service.ts
├── dto/
│   ├── create-[feature].dto.ts
│   ├── update-[feature].dto.ts
│   └── [feature]-query.dto.ts
├── entities/
│   └── [feature].entity.ts
└── tests/
    ├── [feature].controller.spec.ts
    └── [feature].service.spec.ts
```

### Step 4: Implement Entity

```typescript
// src/modules/[feature]/entities/[feature].entity.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  ManyToOne,
  OneToMany,
  JoinColumn,
} from 'typeorm';

@Entity('features')
export class Feature {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'varchar', length: 255 })
  name: string;

  @Column({ type: 'text', nullable: true })
  description: string;

  @Column({ type: 'enum', enum: FeatureStatus, default: FeatureStatus.ACTIVE })
  status: FeatureStatus;

  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  amount: number;

  @Column({ type: 'jsonb', nullable: true })
  metadata: Record<string, any>;

  // Relationships
  @ManyToOne(() => User, (user) => user.features)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ type: 'uuid' })
  userId: string;

  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
}

export enum FeatureStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  PENDING = 'pending',
}
```

### Step 5: Implement DTOs

```typescript
// src/modules/[feature]/dto/create-[feature].dto.ts
import {
  IsString,
  IsNotEmpty,
  IsOptional,
  IsEnum,
  IsNumber,
  IsUUID,
  MinLength,
  MaxLength,
  Min,
  IsEmail,
} from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { Transform } from 'class-transformer';

export class CreateFeatureDto {
  @ApiProperty({ description: 'Feature name', example: 'My Feature' })
  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  @MaxLength(255)
  name: string;

  @ApiPropertyOptional({ description: 'Feature description' })
  @IsString()
  @IsOptional()
  description?: string;

  @ApiProperty({ enum: FeatureStatus, default: FeatureStatus.ACTIVE })
  @IsEnum(FeatureStatus)
  @IsOptional()
  status?: FeatureStatus;

  @ApiProperty({ description: 'Amount in decimal', example: 99.99 })
  @IsNumber({ maxDecimalPlaces: 2 })
  @Min(0)
  @Transform(({ value }) => parseFloat(value))
  amount: number;

  @ApiProperty({ description: 'User ID' })
  @IsUUID()
  userId: string;
}
```

```typescript
// src/modules/[feature]/dto/update-[feature].dto.ts
import { PartialType } from '@nestjs/swagger';
import { CreateFeatureDto } from './create-feature.dto';

export class UpdateFeatureDto extends PartialType(CreateFeatureDto) {}
```

```typescript
// src/modules/[feature]/dto/[feature]-query.dto.ts
import { IsOptional, IsEnum, IsInt, Min, Max } from 'class-validator';
import { Type } from 'class-transformer';
import { ApiPropertyOptional } from '@nestjs/swagger';

export class FeatureQueryDto {
  @ApiPropertyOptional({ default: 1 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiPropertyOptional({ default: 10 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;

  @ApiPropertyOptional({ enum: FeatureStatus })
  @IsOptional()
  @IsEnum(FeatureStatus)
  status?: FeatureStatus;

  @ApiPropertyOptional()
  @IsOptional()
  search?: string;
}
```

### Step 6: Implement Service

```typescript
// src/modules/[feature]/[feature].service.ts
import {
  Injectable,
  NotFoundException,
  BadRequestException,
  ConflictException,
} from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, Like, FindOptionsWhere } from 'typeorm';
import { Feature } from './entities/feature.entity';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';
import { FeatureQueryDto } from './dto/feature-query.dto';
import { PaginatedResult } from '@/common/interfaces/paginated-result.interface';

@Injectable()
export class FeatureService {
  constructor(
    @InjectRepository(Feature)
    private readonly featureRepository: Repository<Feature>,
  ) {}

  async create(createDto: CreateFeatureDto): Promise<Feature> {
    // Business logic validation
    const existing = await this.featureRepository.findOne({
      where: { name: createDto.name, userId: createDto.userId },
    });

    if (existing) {
      throw new ConflictException('Feature with this name already exists');
    }

    const feature = this.featureRepository.create(createDto);
    return this.featureRepository.save(feature);
  }

  async findAll(query: FeatureQueryDto): Promise<PaginatedResult<Feature>> {
    const { page, limit, status, search } = query;
    const skip = (page - 1) * limit;

    const where: FindOptionsWhere<Feature> = {};

    if (status) {
      where.status = status;
    }

    if (search) {
      where.name = Like(`%${search}%`);
    }

    const [data, total] = await this.featureRepository.findAndCount({
      where,
      relations: ['user'],
      skip,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return {
      data,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async findOne(id: string): Promise<Feature> {
    const feature = await this.featureRepository.findOne({
      where: { id },
      relations: ['user'],
    });

    if (!feature) {
      throw new NotFoundException(`Feature with ID ${id} not found`);
    }

    return feature;
  }

  async update(id: string, updateDto: UpdateFeatureDto): Promise<Feature> {
    const feature = await this.findOne(id);

    // Business logic validation
    if (updateDto.name && updateDto.name !== feature.name) {
      const existing = await this.featureRepository.findOne({
        where: { name: updateDto.name, userId: feature.userId },
      });

      if (existing) {
        throw new ConflictException('Feature with this name already exists');
      }
    }

    Object.assign(feature, updateDto);
    return this.featureRepository.save(feature);
  }

  async remove(id: string): Promise<void> {
    const feature = await this.findOne(id);

    // Business logic: check if can be deleted
    // Example: can't delete if has related records
    // if (feature.relatedItems?.length > 0) {
    //   throw new BadRequestException('Cannot delete feature with related items');
    // }

    await this.featureRepository.remove(feature);
  }

  // Additional business methods
  async findByUser(userId: string): Promise<Feature[]> {
    return this.featureRepository.find({
      where: { userId },
      order: { createdAt: 'DESC' },
    });
  }
}
```

### Step 7: Implement Controller

```typescript
// src/modules/[feature]/[feature].controller.ts
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
  Query,
  UseGuards,
  ParseUUIDPipe,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import {
  ApiTags,
  ApiOperation,
  ApiResponse,
  ApiBearerAuth,
  ApiParam,
} from '@nestjs/swagger';
import { JwtAuthGuard } from '@/modules/auth/guards/jwt-auth.guard';
import { RolesGuard } from '@/modules/auth/guards/roles.guard';
import { Roles } from '@/modules/auth/decorators/roles.decorator';
import { CurrentUser } from '@/modules/auth/decorators/current-user.decorator';
import { FeatureService } from './feature.service';
import { CreateFeatureDto } from './dto/create-feature.dto';
import { UpdateFeatureDto } from './dto/update-feature.dto';
import { FeatureQueryDto } from './dto/feature-query.dto';
import { Feature } from './entities/feature.entity';

@ApiTags('Features')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, RolesGuard)
@Controller('api/v1/features')
export class FeatureController {
  constructor(private readonly featureService: FeatureService) {}

  @Post()
  @Roles('admin', 'user')
  @ApiOperation({ summary: 'Create a new feature' })
  @ApiResponse({ status: 201, description: 'Feature created successfully' })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @ApiResponse({ status: 409, description: 'Feature already exists' })
  create(
    @Body() createDto: CreateFeatureDto,
    @CurrentUser() user: User,
  ): Promise<Feature> {
    return this.featureService.create({
      ...createDto,
      userId: user.id,
    });
  }

  @Get()
  @Roles('admin', 'user')
  @ApiOperation({ summary: 'Get all features with pagination' })
  @ApiResponse({ status: 200, description: 'List of features' })
  findAll(@Query() query: FeatureQueryDto) {
    return this.featureService.findAll(query);
  }

  @Get(':id')
  @Roles('admin', 'user')
  @ApiOperation({ summary: 'Get a feature by ID' })
  @ApiParam({ name: 'id', description: 'Feature UUID' })
  @ApiResponse({ status: 200, description: 'Feature found' })
  @ApiResponse({ status: 404, description: 'Feature not found' })
  findOne(@Param('id', ParseUUIDPipe) id: string): Promise<Feature> {
    return this.featureService.findOne(id);
  }

  @Patch(':id')
  @Roles('admin', 'user')
  @ApiOperation({ summary: 'Update a feature' })
  @ApiParam({ name: 'id', description: 'Feature UUID' })
  @ApiResponse({ status: 200, description: 'Feature updated successfully' })
  @ApiResponse({ status: 404, description: 'Feature not found' })
  update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateDto: UpdateFeatureDto,
  ): Promise<Feature> {
    return this.featureService.update(id, updateDto);
  }

  @Delete(':id')
  @Roles('admin')
  @HttpCode(HttpStatus.NO_CONTENT)
  @ApiOperation({ summary: 'Delete a feature' })
  @ApiParam({ name: 'id', description: 'Feature UUID' })
  @ApiResponse({ status: 204, description: 'Feature deleted successfully' })
  @ApiResponse({ status: 404, description: 'Feature not found' })
  remove(@Param('id', ParseUUIDPipe) id: string): Promise<void> {
    return this.featureService.remove(id);
  }
}
```

### Step 8: Implement Module

```typescript
// src/modules/[feature]/[feature].module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { FeatureService } from './feature.service';
import { FeatureController } from './feature.controller';
import { Feature } from './entities/feature.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Feature])],
  controllers: [FeatureController],
  providers: [FeatureService],
  exports: [FeatureService],
})
export class FeatureModule {}
```

### Step 9: Register Module

Add to app.module.ts:

```typescript
// src/app.module.ts
import { FeatureModule } from './modules/feature/feature.module';

@Module({
  imports: [
    // ... other imports
    FeatureModule,
  ],
})
export class AppModule {}
```

### Step 10: Create Migration (if new entity)

```bash
npm run migration:generate -- src/database/migrations/CreateFeatureTable
```

### Step 11: Implement Tests

```typescript
// src/modules/[feature]/tests/[feature].service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { FeatureService } from '../feature.service';
import { Feature } from '../entities/feature.entity';
import { NotFoundException, ConflictException } from '@nestjs/common';

describe('FeatureService', () => {
  let service: FeatureService;
  let repository: Repository<Feature>;

  const mockRepository = {
    create: jest.fn(),
    save: jest.fn(),
    findOne: jest.fn(),
    findAndCount: jest.fn(),
    remove: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        FeatureService,
        {
          provide: getRepositoryToken(Feature),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<FeatureService>(FeatureService);
    repository = module.get<Repository<Feature>>(getRepositoryToken(Feature));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('create', () => {
    it('should create a feature successfully', async () => {
      const createDto = { name: 'Test', userId: 'uuid' };
      const feature = { id: 'uuid', ...createDto };

      mockRepository.findOne.mockResolvedValue(null);
      mockRepository.create.mockReturnValue(feature);
      mockRepository.save.mockResolvedValue(feature);

      const result = await service.create(createDto as any);

      expect(result).toEqual(feature);
      expect(mockRepository.create).toHaveBeenCalledWith(createDto);
    });

    it('should throw ConflictException if feature exists', async () => {
      const createDto = { name: 'Test', userId: 'uuid' };

      mockRepository.findOne.mockResolvedValue({ id: 'existing' });

      await expect(service.create(createDto as any)).rejects.toThrow(
        ConflictException,
      );
    });
  });

  describe('findOne', () => {
    it('should return a feature', async () => {
      const feature = { id: 'uuid', name: 'Test' };
      mockRepository.findOne.mockResolvedValue(feature);

      const result = await service.findOne('uuid');

      expect(result).toEqual(feature);
    });

    it('should throw NotFoundException if not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne('uuid')).rejects.toThrow(NotFoundException);
    });
  });
});
```

---

## Agent Output

Upon completion, the agent should have created:

1. **Entity file** with TypeORM decorators
2. **DTOs** (Create, Update, Query)
3. **Service** with business logic
4. **Controller** with routes and guards
5. **Module** file
6. **Tests** for service and controller
7. **Migration** (if new entity)

And report:
- Created files list
- Endpoint summary
- Any pending TODOs or considerations

---

## Invocation Example

```
Task: API Builder Agent
Prompt: |
  Module: orders
  Endpoints:
    - GET /api/v1/orders - List user orders with pagination
    - POST /api/v1/orders - Create new order
    - GET /api/v1/orders/:id - Get order details
    - PATCH /api/v1/orders/:id/status - Update order status
    - DELETE /api/v1/orders/:id - Cancel order (soft delete)
  Entity: |
    id: uuid (PK)
    userId: uuid (FK to users)
    items: jsonb (array of order items)
    totalAmount: decimal(10,2)
    status: enum (pending, processing, completed, cancelled)
    shippingAddress: jsonb
    createdAt, updatedAt
  Business Logic: |
    - Only order owner or admin can view order
    - Status transitions: pending -> processing -> completed
    - Cannot cancel if status is completed
    - Send email notification on status change
    - Validate items against product inventory
  Context: |
    See projectSpec section 3.5 for order schema
    Related to: users, products modules
```

---

## Important Notes

- **Follow NestJS conventions** - module structure, dependency injection
- **Always validate DTOs** - use class-validator decorators
- **Document with Swagger** - ApiProperty, ApiOperation, ApiResponse
- **Implement guards** - JwtAuthGuard, RolesGuard for protected routes
- **Handle errors properly** - use NestJS exceptions
- **Write tests** - at minimum service unit tests
- **Create migrations** - for database schema changes
- **Use transactions** - for operations affecting multiple tables
