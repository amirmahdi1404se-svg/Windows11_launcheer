package com.win11.standalone;

import android.annotation.SuppressLint;
import android.content.Intent;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.Environment;
import android.provider.Settings;
import android.view.View;
import android.webkit.JavascriptInterface;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONArray;
import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Map;

public class MainActivity extends AppCompatActivity {

    private WebView webView;
    private boolean isEngineReady = false;
    private String engineStatusText = "در حال آماده‌سازی هسته ویندوز ۱۱...";

    @SuppressLint("SetJavaScriptEnabled")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);

        requestStoragePermission();

        webView = new WebView(this);
        setContentView(webView);

        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setAllowFileAccess(true);
        settings.setAllowContentAccess(true);

        webView.addJavascriptInterface(new Win11Bridge(), "Win11Native");
        webView.setWebViewClient(new WebViewClient());
        webView.loadUrl("file:///android_asset/index.html");

        setupInternalEngine();
    }

    private void requestStoragePermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            if (!Environment.isExternalStorageManager()) {
                try {
                    Intent intent = new Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION);
                    intent.setData(Uri.parse("package:" + getPackageName()));
                    startActivity(intent);
                } catch (Exception ignored) {}
            }
        }
    }

    private void setupInternalEngine() {
        new Thread(() -> {
            try {
                File baseDir = getFilesDir();
                File binDir = new File(baseDir, "usr/bin");
                File wineDir = new File(baseDir, "usr/wine");
                File prefixDir = new File(baseDir, "prefix");

                if (!binDir.exists()) binDir.mkdirs();
                if (!wineDir.exists()) wineDir.mkdirs();
                if (!prefixDir.exists()) prefixDir.mkdirs();

                File box64File = new File(binDir, "box64");
                File wineArchive = new File(baseDir, "wine.tar.xz");
                File wineBinary = new File(wineDir, "bin/wine");

                if (!box64File.exists() || box64File.length() == 0) {
                    engineStatusText = "در حال نصب مترجم پردازنده Box64...";
                    copyAsset("engine/box64", box64File);
                    box64File.setExecutable(true, false);
                }

                if (!wineBinary.exists()) {
                    engineStatusText = "در حال استخراج هسته Wine...";
                    copyAsset("engine/wine.tar.xz", wineArchive);
                    if (wineArchive.length() > 1000) {
                        Process tarProc = new ProcessBuilder(
                                "tar", "-xf", wineArchive.getAbsolutePath(),
                                "-C", wineDir.getAbsolutePath(), "--strip-components=1"
                        ).start();
                        tarProc.waitFor();
                        wineArchive.delete();
                    }
                    File wineBinFolder = new File(wineDir, "bin");
                    if (wineBinFolder.exists() && wineBinFolder.listFiles() != null) {
                        for (File f : wineBinFolder.listFiles()) {
                            f.setExecutable(true, false);
                        }
                    }
                }

                isEngineReady = true;
                engineStatusText = "آماده به کار (Box64 + Wine Active)";

            } catch (Exception e) {
                is

