# Backend Documentation

## 🏗️ Architecture & Infrastructure

### [1. Architecture](./1-architecture.md)
- NestJS (TypeScript) setup
- REST API Server configuration
- TypeORM integration
- Package dependencies

### [2. Database Schema](./2-database.md)
- Database tables structure
- Entity relationships
- Field definitions
- Indexing strategy

### [3. API Routes](./3-api-routes.md)
- REST API endpoint definitions
- Endpoint specifications
- Request/response formats
- Error handling

## 💻 Development

### [4. Development Process](./4-development.md)
- Setup development environment
- Coding standards
- Git workflow
- Code review process

### [5. Testing](./5-testing.md)
- Unit testing with Jest
- Integration testing
- Test coverage requirements
- Testing tools setup

### [6. Deployment](./6-deployment.md)
- Environment configuration
- CI/CD pipeline
- Production deployment
- Monitoring setup

## 🚀 Quick Start

1. **Setup Environment**
   ```bash
   cd apps/backend
   npm install
   ```

2. **Database Setup**
   ```bash
   # Setup and run TypeORM migrations
   npm run typeorm migration:run
   ```

3. **Run Development Server**
   ```bash
   npm run dev
   ```

## 🔗 Cross-References

- **Frontend Integration** → [Frontend Documentation](../frontend/)
- **Shared Features** → [Shared Documentation](../shared/)
- **Database Schema** → [Database Documentation](./2-database.md)
- **API Documentation** → [API Routes](./3-api-routes.md)

## 📝 Development Notes

- Menggunakan NestJS dengan TypeScript
- REST API untuk type-safe endpoints
- TypeORM sebagai ORM
- PostgreSQL sebagai database
- Jest untuk testing
- GitHub Actions untuk CI/CD
