# TCX Reducer

A web application for reducing TCX file size by thinning out GPS trackpoints.

## Overview

This tool reduces TCX (Training Center XML) file size by skipping a specified number of trackpoints. It's useful when you need to reduce the file size of detailed track data exported from GPS devices.

**Live Demo**: [https://tcx-reducer.fwa.workers.dev/](https://tcx-reducer.fwa.workers.dev/)

## Features

- ✅ Drag & drop support
- ✅ Display file size and record count
- ✅ Instant thinning processing
- ✅ Responsive design (mobile-friendly)
- ✅ Client-side only (no server required)
- ✅ Security measures implemented

## Usage

### Online (Recommended)

Visit the deployed application at: https://tcx-reducer.fwa.workers.dev/

### Local Development

1. Clone this repository
2. Open `public/index.html` in your browser, or
3. Run `npx wrangler pages dev public` for local development server

## How It Works

Given a thinning number `n`, the tool keeps 1 record for every `n+1` records.

Examples:
- Thinning = 1: Keeps 50% of original records
- Thinning = 2: Keeps 33% of original records
- Thinning = 3: Keeps 25% of original records

## Limitations

- Maximum file size: 10MB
- Thinning number range: 1-100
- Supported format: TCX (Training Center XML)

## What is TCX?

TCX (Training Center XML) is an XML-based file format used by fitness devices such as Garmin. It contains training data including GPS position, heart rate, cadence, and more.

## Security

This application implements the following security measures:

- ✅ File size limit (10MB)
- ✅ Filename validation (path traversal prevention)
- ✅ File type validation
- ✅ XML parse error handling
- ✅ Input value range checking
- ✅ HTML escaping
- ✅ Client-side processing only (no server upload)

## Browser Requirements

- Modern browsers (latest versions of Chrome, Firefox, Safari, Edge)
- JavaScript enabled

## Deployment

This application is deployed on **Cloudflare Pages**.

To deploy your own instance:

```bash
npm install -g wrangler
wrangler pages deploy public
```
