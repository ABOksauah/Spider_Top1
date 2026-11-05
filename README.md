
name: Build Spider APK

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3

      - name: Install SDK
        run: |
          yes | sdkmanager --licenses
          sdkmanager "platforms;android-35" "build-tools;35.0.0"

      - name: Extract Source
        run: |
          unzip -q Spider_Top1_REPAIRED.zip -d source

      - name: Build APK
        run: |
          GRADLEW=$(find source -name gradlew -type f | head -n 1)
          if [ -z "$GRADLEW" ]; then
            echo "gradlew not found"
            exit 1
          fi
          chmod +x "$GRADLEW"
          cd "$(dirname "$GRADLEW")"
          ./gradlew --no-daemon assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Spider-APK
          path: source/**/build/outputs/apk/debug/*.apk
          
