---
title: Rust项目配置Github Action
subtitle:
date: 2024-06-20T23:35:24+08:00
draft: false
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - Github Action
categories:
  - Rust
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---

<!--more-->
# Rust 项目配置 Github Action

## 多平台编译的几种方式

### 1. 在多个平台上分别编译
```yaml
name: cross-release
on:
    workflow_dispatch:
    push:
      tags:
        - '**'
env:
  # The project name specified in your Cargo.toml
  PROJECT_NAME: action-demo

jobs:
  release:
    runs-on: ${{ matrix.runner }}

    strategy:
      matrix:
        # You can add more, for any target you'd like!
        include:
          - name: linux-amd64-gnu
            runner: ubuntu-latest
            target: x86_64-unknown-linux-gnu
          - name: linux-amd64-musl
            runner: ubuntu-latest
            target: x86_64-unknown-linux-musl
          - name: macos-amd64
            runner: macos-latest
            target: x86_64-apple-darwin
          - name: macos-arm64
            runner: macos-latest
            target: aarch64-apple-darwin
          - name: windows-amd64
            runner: windows-latest
            target: x86_64-pc-windows-msvc
          - name: windows-gnu
            runner: windows-latest
            target: x86_64-pc-windows-gnu
    steps:

      - name: Checkout
        uses: actions/checkout@v3

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: "${{ matrix.target }}"

      - name: Build Binary
        run: cargo build --verbose --locked --release --target ${{ matrix.target }}

      - name: Release Binary
        shell: bash
        run: |
          BIN_SUFFIX=""
          if [[ "${{ matrix.runner }}" == "windows-latest" ]]; then
            BIN_SUFFIX=".exe"
          fi
          ls target/*
          # The built binary output location
          BIN_OUTPUT="target/${{ matrix.target }}/release/${PROJECT_NAME}${BIN_SUFFIX}"

          # Define a better name for the final binary
          BIN_RELEASE="${PROJECT_NAME}-${{ matrix.name }}${BIN_SUFFIX}"
          BIN_RELEASE_VERSIONED="${PROJECT_NAME}-${{ github.ref_name }}-${{ matrix.name }}${BIN_SUFFIX}"
          tar -zcf "${PROJECT_NAME}.tar.gz" "${BIN_OUTPUT}"

          # Move the built binary where you want it
          # mv "${BIN_OUTPUT}" "./<your-destination>/${BIN_RELEASE}"

      - name : upload binary
        uses: actions/upload-artifact@master
        if: always()
        with:
          name: ${{ matrix.name }}
          path: ./*.tar.gz
```