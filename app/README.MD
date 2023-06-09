## AYUDA TIte


name: CI
on:
  push:
    branches:
      - main
jobs:
  flutter_test:
    name: Run flutter test and analize
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: "12.x"
      - uses: subosito/flutter-action@v1
        with:
          channel: "stable"
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test
      
  build_ios: 
    name: Build Flutter (iOS)
    needs: flutter_test
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: "12.x"
      - uses: subosito/flutter-action@v1
        with:
          channel: "stable"
      - run: flutter pub get
      - run: flutter clean
      - run: flutter build ios --release
      - name: Upload iPA
        uses: actions/upload-artifact@v2
        with:
          name: ios-build
          path: build/app/ios-relase.app
      
  build_appbundle: 
    name: Build Flutter (Android)
    runs-on: ubuntu-latest
    needs: flutter_test
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: "12.x"
      - uses: subosito/flutter-action@v1
        with:
          channel: "stable"
      - run: flutter pub get
      - run: flutter clean
      - run: flutter build appbundle

      - uses: actions/upload-artifact@master
        with:
          name: apk-build
          path: ./app/app-relase-${{ github.sha }}.apk
      
  relase:
    name: Create a Release APK
    runs-on: ubuntu-latest
    needs: [build_appbundle]
    env: 
      GITHUB_TOKEN: ${{ secrets.TOKEN }}
    steps:
      - name: Create Release
        id: create_release
        uses: actions/create-release@v1.0.0
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Changes in this Release
            - First Change
            - Second Change
          draft: false
          prerelease: false
      - name: Upload Release Asset
        id: upload-release-asset
        uses: actions/upload-release-asset@v1.0.1
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
        with:
          asset_path: ./app/*.apk
          asset_name: app-release-${{ github.sha }}.apk
          asset_content_type: application/vnd.android.package-archive
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          release_id: ${{ steps.create_release.outputs.id }}
          github_token: $GITHUB_TOKEN