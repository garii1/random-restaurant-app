# random-restaurant-appname: Build Android APK

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        
    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        
    - name: Install Capacitor
      run: npm install -g @capacitor/cli
      
    - name: Create package.json
      run: |
        cat > package.json << 'EOF'
        {
          "name": "random-restaurant-app",
          "version": "1.0.0",
          "dependencies": {
            "@capacitor/android": "^6.0.0",
            "@capacitor/core": "^6.0.0"
          }
        }
        EOF
        
    - name: Install dependencies
      run: npm install
      
    - name: Create Capacitor config
      run: |
        cat > capacitor.config.ts << 'EOF'
        import type { CapacitorConfig } from "@capacitor/cli";
        const config: CapacitorConfig = {
          appId: "com.randomrestaurant.app",
          appName: "오늘은 랜덤 식사",
          webDir: "dist"
        };
        export default config;
        EOF
        
    - name: Create web files
      run: |
        mkdir -p dist
        cat > dist/index.html << 'EOF'
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="utf-8">
          <title>오늘은 랜덤 식사</title>
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <style>
            body { 
              font-family: Arial, sans-serif; 
              text-align: center; 
              padding: 50px; 
              background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
              color: white;
              margin: 0;
              min-height: 100vh;
              display: flex;
              flex-direction: column;
              justify-content: center;
            }
            h1 { 
              color: white; 
              font-size: 2.5em; 
              margin: 20px 0;
              text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            }
            .emoji { 
              font-size: 4em; 
              margin: 20px; 
              animation: bounce 2s infinite;
            }
            p {
              font-size: 1.2em;
              margin: 10px 0;
              opacity: 0.9;
            }
            @keyframes bounce {
              0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
              40% { transform: translateY(-30px); }
              60% { transform: translateY(-15px); }
            }
            .features {
              margin-top: 30px;
              font-size: 1.1em;
            }
            .feature {
              margin: 10px 0;
              padding: 10px;
              background: rgba(255,255,255,0.1);
              border-radius: 10px;
            }
          </style>
        </head>
        <body>
          <div class="emoji">🍽️</div>
          <h1>오늘은 랜덤 식사</h1>
          <p>전국 맛집 추천 안드로이드 앱</p>
          <div class="features">
            <div class="feature">🎲 룰렛 방식 랜덤 선택</div>
            <div class="feature">📍 GPS 기반 실시간 검색</div>
            <div class="feature">⭐ 리뷰 및 즐겨찾기</div>
            <div class="feature">🎮 게임화된 UI/UX</div>
          </div>
          <div class="emoji">🎲</div>
        </body>
        </html>
        EOF
        
    - name: Initialize Android
      run: |
        npx cap add android
        npx cap sync android
        
    - name: Build APK
      run: |
        cd android
        chmod +x gradlew
        ./gradlew assembleDebug
        
    - name: Upload APK
      uses: actions/upload-artifact@v4
      with:
        name: random-restaurant-apk
        path: android/app/build/outputs/apk/debug/app-debug.apk
