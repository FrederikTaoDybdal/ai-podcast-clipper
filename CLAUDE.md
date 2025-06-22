# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Podcast Clipper SaaS - A full-stack application that automatically converts long-form podcasts into viral short-form clips for platforms like YouTube Shorts and TikTok. The system uses AI models for transcription, active speaker detection, and viral moment identification.

## Architecture

### Frontend (ai-podcast-clipper-frontend/)
- **Stack**: Next.js 15, React 19, TypeScript, Tailwind CSS v4, Shadcn UI
- **Auth**: NextAuth.js (Auth.js) with Prisma adapter
- **Database**: PostgreSQL with Prisma ORM
- **Payments**: Stripe for credit-based billing
- **File Storage**: AWS S3 with presigned URLs
- **Background Jobs**: Inngest for queue management

### Backend (ai-podcast-clipper-backend/)
- **Framework**: FastAPI (Python 3.12)
- **Deployment**: Modal (serverless GPU platform)
- **AI Models**:
  - WhisperX for transcription
  - LR-ASD (Junhua-Liao) for active speaker detection
  - Gemini 2.5 Pro for viral moment identification
- **Video Processing**: FFmpegCV with GPU acceleration

## Essential Development Commands

### Frontend Development
```bash
# Development server
npm run dev

# Queue worker (run in separate terminal)
npm run inngest-dev

# Database management
npm run db:push        # Push schema changes
npm run db:studio      # Open Prisma Studio
npm run db:generate    # Generate Prisma client
npm run db:migrate     # Run migrations

# Code quality
npm run lint           # Run ESLint
npm run lint:fix       # Fix ESLint issues
npm run typecheck      # TypeScript checks
npm run format:write   # Format with Prettier

# Build and deploy
npm run build          # Production build
npm run start          # Start production server
```

### Backend Development
```bash
# Local development (requires Modal account)
modal run main.py

# Deploy to production
modal deploy main.py
```

## Project Structure

### Key Frontend Directories
- `src/app/` - Next.js app router pages and layouts
- `src/server/` - Server-side code including:
  - `api/` - tRPC routers
  - `auth.ts` - NextAuth configuration
  - `db.ts` - Prisma client
- `src/components/` - React components
- `src/inngest/` - Background job functions
- `src/lib/` - Utility functions and helpers
- `prisma/` - Database schema and migrations

### Key Backend Files
- `main.py` - Main FastAPI application and Modal deployment
- `viral_identifier.py` - Gemini AI integration for viral moment detection
- `requirements.txt` - Python dependencies

## Environment Configuration

The frontend requires these environment variables (defined in `src/env.js`):

### Required Server Variables
- `AUTH_SECRET` - NextAuth secret
- `DATABASE_URL` - PostgreSQL connection string (not SQLite despite .env.example)
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` - AWS credentials
- `S3_BUCKET_NAME` - S3 bucket for video storage
- `STRIPE_SECRET_KEY` - Stripe API key
- `STRIPE_WEBHOOK_SECRET` - Stripe webhook endpoint secret
- `STRIPE_SMALL_PACK_PRICE_ID`, `STRIPE_MEDIUM_PACK_PRICE_ID`, `STRIPE_LARGE_PACK_PRICE_ID` - Stripe product IDs
- `MODAL_ENDPOINT` - Modal serverless function URL

### Required Client Variables
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` - Stripe public key

## Database Schema

Key models in `prisma/schema.prisma`:
- `User` - Contains credits, stripeCustomerId
- `UploadedFile` - Original videos with processing status
- `Clip` - Generated clips linked to uploads and users

## Processing Pipeline

1. User uploads video → S3 storage
2. Inngest triggers `processVideo` function
3. Modal serverless function processes:
   - Downloads from S3
   - Transcribes with WhisperX
   - Detects speakers with LR-ASD
   - Identifies viral moments with Gemini
   - Creates vertical crops with FFmpeg
4. Clips saved to S3, database updated

## Important Implementation Details

### Credit System
- Users start with 10 free credits
- 1 credit per clip generated
- Stripe integration for purchasing credit packs
- Credit checking in `processVideo` function before processing

### File Upload Flow
- Client requests presigned URL from `/api/upload`
- Direct upload to S3 from browser
- Database record created after successful upload
- Background processing triggered via Inngest

### Modal Deployment
- Backend requires `asd` folder (LR-ASD repository) to be present
- GPU-enabled processing for video tasks
- Serverless scaling with Modal platform

## Common Development Tasks

### Adding New API Endpoints
1. Frontend: Add tRPC router in `src/server/api/routers/`
2. Backend: Add FastAPI endpoint in `main.py`

### Modifying Database Schema
1. Edit `prisma/schema.prisma`
2. Run `npm run db:push` for development
3. Run `npm run db:migrate` for production

### Testing Video Processing
1. Ensure Modal is configured: `modal token set`
2. Start frontend: `npm run dev`
3. Start Inngest: `npm run inngest-dev`
4. Upload test video through UI

## AWS S3 Configuration

S3 bucket requires specific CORS policy (see README.md for full configuration):
- Allowed origins: frontend URLs
- Allowed methods: GET, PUT, POST
- Exposed headers for upload progress tracking