FROM python:3.10.12-bookworm

ARG BASE_COMMIT=d987297b012fe90ddfff9efcc524628cd53b4d25

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV PYVISTA_OFF_SCREEN=true

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates git patch \
    libgl1-mesa-glx libglu1-mesa xvfb \
 && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL "https://codeload.github.com/pyvista/pyvista/tar.gz/${BASE_COMMIT}" \
      -o /tmp/src.tgz \
 && tar tzf /tmp/src.tgz | head -1 | grep -qF "pyvista-${BASE_COMMIT}/" \
 && mkdir -p /app \
 && tar xzf /tmp/src.tgz --strip-components=1 -C /app \
 && rm /tmp/src.tgz

WORKDIR /app

RUN pip install --no-cache-dir -e ".[colormaps]" \
 && pip install --no-cache-dir pytest matplotlib hypothesis

# Conftest exit hook v2 (D11)
RUN printf 'import os as _jf_os, sys as _jf_sys, pytest as _jf_pytest\n\
_JF_EXIT = {"status": 0}\n\
@_jf_pytest.hookimpl(trylast=True)\n\
def pytest_sessionfinish(session, exitstatus):\n\
    _JF_EXIT["status"] = int(exitstatus)\n\
@_jf_pytest.hookimpl(trylast=True)\n\
def pytest_unconfigure(config):\n\
    _jf_sys.stdout.flush(); _jf_sys.stderr.flush()\n\
    _jf_os._exit(_JF_EXIT["status"])\n' >> /app/conftest.py

RUN python -m pytest tests/plotting/test_colors.py tests/plotting/test_theme.py -v -o addopts= -W default -k test_cmap
