# Airbnb Clone - Frontend

A fully functional Airbnb clone built with Next.js and Sanity.io for content management. This project demonstrates modern web development practices with structured content, real-time data fetching, and interactive map features.

## 🎥 Tutorial

This repository supports the complete tutorial on building an AirBnb Clone with structured content. Watch the full tutorial [here](https://youtu.be/mx1dbMzd3tU).

## ✨ Features

- 🏠 **Property Listings**: Browse available properties with detailed information
- 🗺️ **Interactive Maps**: Google Maps integration with custom markers for property locations
- 📍 **Geolocation**: Display properties on an interactive map dashboard
- ⭐ **Reviews System**: User reviews with ratings and traveller information
- 🖼️ **Image Gallery**: Property image galleries with optimized loading
- 📱 **Responsive Design**: Mobile-friendly interface
- ⚡ **Server-Side Rendering**: Fast page loads with Next.js SSR
- 🎨 **Modern UI**: Clean and intuitive user interface

## 🛠️ Tech Stack

- **[Next.js](https://nextjs.org/)** (v10.1.3) - React framework for production
- **[React](https://reactjs.org/)** (v17.0.2) - UI library
- **[Sanity.io](https://www.sanity.io/)** - Headless CMS for content management
- **[Google Maps API](https://developers.google.com/maps)** - Interactive maps and markers
- **next-sanity** - Official Sanity.io toolkit for Next.js
- **@react-google-maps/api** - React components for Google Maps

## 📁 Project Structure

```
├── components/
│   ├── DashboardMap.js    # Main map with clustered property markers
│   ├── Map.js             # Individual property location map
│   ├── Image.js           # Optimized image component
│   ├── NavBar.js          # Navigation bar component
│   └── Review.js          # Review display component
├── pages/
│   ├── _app.js            # Next.js app wrapper
│   ├── index.js           # Home page with property listings
│   ├── property/
│   │   └── [slug].js      # Dynamic property detail page
│   └── api/               # API routes
├── styles/
│   └── globals.css        # Global styles
├── public/                # Static assets
├── images/                # Image assets
├── sanity.js              # Sanity client configuration
├── utils.js               # Utility functions
└── package.json           # Project dependencies
```

## 🚀 Getting Started

### Prerequisites

- Node.js (v12 or higher)
- npm or yarn
- A Sanity.io account and project
- Google Maps API key

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/kubowania/airbnb-sanity-frontend.git
   cd airbnb-sanity-frontend
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   yarn install
   ```

3. **Set up environment variables**
   
   Create a `.env.local` file in the root directory:
   ```env
   NEXT_PUBLIC_SANITY_PROJECT_ID=your_project_id
   NEXT_PUBLIC_SANITY_DATASET=production
   googlePlacesAPI=your_google_maps_api_key
   ```

4. **Set up Sanity Backend**
   
   You'll need to set up the Sanity backend. Find the backend repository [here](https://github.com/kubowania/airbnb-sanity-backend).

5. **Run the development server**
   ```bash
   npm run dev
   # or
   yarn dev
   ```

6. **Open your browser**
   
   Navigate to [http://localhost:3000](http://localhost:3000)

## 📝 Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm start` - Start production server

## 🔧 Configuration

### Sanity.io Setup

The project uses Sanity.io for content management. Configure your Sanity client in `sanity.js`:

- Set your project ID
- Configure dataset (production/development)
- Enable CDN for production

### Google Maps API

To enable map features:

1. Get a Google Maps API key from [Google Cloud Console](https://console.cloud.google.com/)
2. Enable Maps JavaScript API
3. Add the API key to your environment variables

## 📚 Key Concepts Covered

- **Sanity Schemas & Types**: Define structured content models
- **Content Management Systems**: Manage data visually with Sanity Studio
- **Geolocation**: Work with geographic coordinates
- **Google Maps Integration**: Display interactive maps with custom markers
- **Marker Clustering**: Efficiently display multiple property locations
- **Next.js Server-Side Props**: Fetch data at request time
- **GROQ Queries**: Query Sanity content with GROQ
- **React Hooks**: Modern React patterns
- **Dynamic Routing**: Create dynamic property pages
- **Image Optimization**: Optimize images with Sanity's image pipeline

## 📖 Tutorial Timeline

- 0:00 - Introduction
- 2:39 - Getting started with Sanity
- 11:43 - Creating Sanity Schemas
- 24:55 - Creating PropertyImage Type
- 30:15 - Creating Review Schema
- 34:44 - Creating Traveller Schema
- 35:50 - Creating Person Schema
- 42:28 - Adding Data to Sanity Studio
- 52:17 - Getting Started with Next.js
- 56:00 - Connecting Sanity to Next.js
- 1:00:20 - Querying Data with GROQ
- 1:03:57 - Building Next.js pages
- 1:24:23 - Styling with CSS and Google Fonts
- 1:31:10 - Adding images
- 1:43:25 - Property information styling
- 1:46:45 - Review information
- 1:54:25 - Location information
- 2:06:01 - Adding Links with Next.js
- 2:07:15 - Building the home page
- 2:17:40 - Building NavBar component
- 2:21:30 - Adding the Airbnb logo
- 2:23:35 - Adding a Cluster Map
- 2:30:25 - Conclusion

## 🔗 Related Resources

- [Sanity Backend Repository](https://github.com/kubowania/airbnb-sanity-backend)
- [Google Maps API Documentation](https://developers.google.com/maps)
- [Sanity.io Documentation](https://www.sanity.io/docs)
- [Next.js Documentation](https://nextjs.org/docs)
- [Sanity Community Exchange](https://www.sanity.io/exchange)
- [Sanity Slack Community](https://slack.sanity.io/)

## 🤝 Support

If you get stuck, the Sanity community can help:
- [Sanity Exchange](https://www.sanity.io/exchange)
- [Sanity Slack Community](https://slack.sanity.io/)

## 👩‍💻 Author

**Ania Kubów**

- [YouTube Tutorial](https://youtu.be/mx1dbMzd3tU)
- [Buy me a coffee](https://www.buymeacoffee.com/aniakubow)
- [Newsletter - Weekly coding tips](https://bit.ly/JS-tips)

## 🙏 Acknowledgments

- Tutorial uses [Tabnine](http://bit.ly/tabnine-top-tool) for AI-powered code completion
- Built with the Sanity.io community support

## 📄 License

This project is open source and available for educational purposes.

---

⭐ If you found this project helpful, please consider giving it a star!
