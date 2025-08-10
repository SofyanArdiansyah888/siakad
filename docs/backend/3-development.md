# Development Process

1. Setup backend dengan NestJS, REST API, dan TypeORM
2. Testing unit & e2e (lihat [docs/backend/5-testing.md](5-testing.md))
3. Deploy ke staging (lihat [docs/backend/6-deployment.md](6-deployment.md))
4. Deploy ke production (lihat [docs/backend/6-deployment.md](6-deployment.md))

## 🏗️ NestJS Best Practices

### Architecture Patterns

#### 1. Module Organization
```typescript
// Feature-based module structure
src/
├── modules/
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.guard.ts
│   │   └── dto/
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   └── entities/
│   └── shared/
│       ├── shared.module.ts
│       ├── guards/
│       ├── interceptors/
│       └── pipes/
```

#### 2. Dependency Injection
```typescript
// ✅ Good: Use constructor injection
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly configService: ConfigService,
  ) {}
}

// ❌ Bad: Don't use property injection
@Injectable()
export class UserService {
  @Inject(UserRepository)
  private userRepository: UserRepository;
}
```

#### 3. Service Layer Pattern
```typescript
// Controller should be thin, delegate to service
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}

// Service handles business logic
@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    // Business logic here
    return this.userRepository.findOne(id);
  }
}
```

### Coding Standards

#### 1. Naming Conventions
- **Controllers**: `*.controller.ts` (e.g., `users.controller.ts`)
- **Services**: `*.service.ts` (e.g., `users.service.ts`)
- **Modules**: `*.module.ts` (e.g., `users.module.ts`)
- **DTOs**: `*.dto.ts` (e.g., `create-user.dto.ts`)
- **Entities**: `*.entity.ts` (e.g., `user.entity.ts`)
- **Guards**: `*.guard.ts` (e.g., `auth.guard.ts`)
- **Interceptors**: `*.interceptor.ts` (e.g., `logging.interceptor.ts`)

#### 2. File Structure
```typescript
// ✅ Good: Consistent file structure
export class UsersController {
  // 1. Constructor
  constructor(private readonly usersService: UsersService) {}

  // 2. Public methods (GET, POST, PUT, DELETE)
  @Get()
  async findAll() {}

  @Post()
  async create() {}

  // 3. Private helper methods
  private validateUser() {}
}
```

#### 3. Error Handling
```typescript
// Use built-in exception filters
@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string) {
    const user = await this.usersService.findOne(id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }
}

// Custom exception filter
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message: exception.message,
    });
  }
}
```

### Data Transfer Objects (DTOs)

#### 1. Input Validation
```typescript
// Use class-validator decorators
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @Length(2, 50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  password: string;
}

// Apply validation pipe globally
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));
```

#### 2. Response DTOs
```typescript
// Define response schemas
export class UserResponseDto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  name: string;

  @ApiProperty()
  email: string;

  @ApiProperty()
  createdAt: Date;
}
```

### Guards and Authentication

#### 1. JWT Authentication
```typescript
// JWT Strategy
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}

// Auth Guard
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

#### 2. Role-based Authorization
```typescript
// Custom decorator
export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// Role guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) {
      return true;
    }
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}
```

### Interceptors and Middleware

#### 1. Logging Interceptor
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const method = request.method;
    const url = request.url;
    const now = Date.now();

    return next.handle().pipe(
      tap(() => {
        console.log(`${method} ${url} ${Date.now() - now}ms`);
      }),
    );
  }
}
```

#### 2. Transform Interceptor
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

### Configuration Management

#### 1. Environment Configuration
```typescript
// config/configuration.ts
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  },
});

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration],
    }),
  ],
})
export class AppModule {}
```

#### 2. Validation Schema
```typescript
// config/env.validation.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
  PORT: Joi.number().default(3000),
  DATABASE_HOST: Joi.string().required(),
  DATABASE_PORT: Joi.number().default(5432),
  JWT_SECRET: Joi.string().required(),
});
```

### Testing Best Practices

#### 1. Unit Testing
```typescript
describe('UsersService', () => {
  let service: UsersService;
  let repository: UserRepository;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UserRepository,
          useValue: {
            findOne: jest.fn(),
            create: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<UserRepository>(UserRepository);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

#### 2. E2E Testing
```typescript
describe('Users (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200);
  });
});
```

### Performance Optimization

#### 1. Caching
```typescript
// Use cache interceptor
@Controller('users')
@UseInterceptors(CacheInterceptor)
export class UsersController {
  @Get(':id')
  @CacheKey('user')
  @CacheTTL(30)
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

#### 2. Database Optimization
```typescript
// Use query optimization
@Injectable()
export class UsersService {
  async findAll(): Promise<User[]> {
    return this.userRepository.find({
      select: ['id', 'name', 'email'], // Select only needed fields
      relations: ['profile'], // Eager load relations
      order: { createdAt: 'DESC' },
    });
  }
}
```

### Security Best Practices

#### 1. Input Sanitization
```typescript
// Use built-in sanitization
@Post()
async create(@Body() createUserDto: CreateUserDto) {
  // Validation pipe automatically sanitizes input
  return this.usersService.create(createUserDto);
}
```

#### 2. Rate Limiting
```typescript
// Apply rate limiting
@Controller('auth')
@UseGuards(ThrottlerGuard)
export class AuthController {
  @Post('login')
  @Throttle(5, 60) // 5 requests per minute
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }
}
```

### Code Quality

#### 1. ESLint Configuration
```json
{
  "extends": [
    "@nestjs/eslint-config",
    "@nestjs/eslint-config/dist/configs/type-checked"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn"
  }
}
```

#### 2. Pre-commit Hooks
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

### Documentation

#### 1. API Documentation with Swagger
```typescript
// main.ts
const config = new DocumentBuilder()
  .setTitle('SIAKAD API')
  .setDescription('Sistem Informasi Akademik API')
  .setVersion('1.0')
  .addBearerAuth()
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api', app, document);
```

#### 2. Code Documentation
```typescript
/**
 * Service untuk mengelola data user
 * @description Menangani operasi CRUD untuk user
 */
@Injectable()
export class UsersService {
  /**
   * Mencari user berdasarkan ID
   * @param id - ID user yang dicari
   * @returns Promise<User> - Data user atau null jika tidak ditemukan
   */
  async findOne(id: string): Promise<User> {
    return this.userRepository.findOne(id);
  }
}
```
