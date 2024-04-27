---
layout: post
date: 2024-4-26
title: Setting up Nextcloud on NixOS
---

# NixOS and Modularity

NixOS is a very incredible Linux distribution, and everything I've read has convinced me of its usefulness. This is especially true when you consider the problems that this distro has been set out to solve. One of the best-known facts of NixOS--and my personal favorite feature--is that you can declare your system.

Part of NixOS's declarative nature allows you to create modules. Modules are small subsets or snippets of configuration that can be toggled on or off, copied/pasted between different machines, and allow for high levels of customization and modularity (hence the name) between systems and builds.

On that note, I wanted to explore my most recent NixOS module adventure. I had some (self inflicted) difficulties with this project that turned out to be very educational.

# Getting Started

Assuming that the user has some familiarity with the configuration.nix file, modules, and how those concepts relate, I will now dive into setting up a nextcloud.nix module!

Initially, I wanted to get started with this project because of Jupiter Broadcasting's efforts for their [Nixcloud](https://github.com/JupiterBroadcasting/nixconfigs/blob/main/nextcloud.nix) project. However, after getting into it, I realized that I didn't necessarily want this preset. The reason being that I didn't really intend to expose my Nextcloud instance to the internet, and would instead access it only via a VPN.

Regardless, for the sake of easy viewing, here is the JB nextcloud.nix configuration:
```
{ self, config, lib, pkgs, ... }: {
  # Based on https://carjorvaz.com/posts/the-holy-grail-nextcloud-setup-made-easy-by-nixos/
  security.acme = {
    acceptTerms = true;
    defaults = {
      email = "wes+barn-acme@jupiterbroadcasting.com";
      dnsProvider = "cloudflare";
      # location of your CLOUDFLARE_DNS_API_TOKEN=[value]
      # https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html#EnvironmentFile=
      environmentFile = "/REPLACE/WITH/YOUR/PATH";
    };
  };
  services = {
    nginx.virtualHosts = {
      "YOUR.DOMAIN.NAME" = {
        forceSSL = true;
        enableACME = true;
        # Use DNS Challenege.
        acmeRoot = null;
      };
    };
    #
    nextcloud = {
      enable = true;
      hostName = "YOUR.DOMAIN.NAME";
      # Need to manually increment with every major upgrade.
      package = pkgs.nextcloud28;
      # Let NixOS install and configure the database automatically.
      database.createLocally = true;
      # Let NixOS install and configure Redis caching automatically.
      configureRedis = true;
      # Increase the maximum file upload size.
      maxUploadSize = "16G";
      https = true;
      autoUpdateApps.enable = true;
      extraAppsEnable = true;
      extraApps = with config.services.nextcloud.package.packages.apps; {
        # List of apps we want to install and are already packaged in
        # https://github.com/NixOS/nixpkgs/blob/master/pkgs/servers/nextcloud/packages/nextcloud-apps.json
        inherit calendar contacts notes onlyoffice tasks cookbook qownnotesapi;
        # Custom app example.
        socialsharing_telegram = pkgs.fetchNextcloudApp rec {
          url =
            "https://github.com/nextcloud-releases/socialsharing/releases/download/v3.0.1/socialsharing_telegram-v3.0.1.tar.gz";
          license = "agpl3";
          sha256 = "sha256-8XyOslMmzxmX2QsVzYzIJKNw6rVWJ7uDhU1jaKJ0Q8k=";
        };
      };
      config = {
        overwriteProtocol = "https";
        defaultPhoneRegion = "US";
        dbtype = "pgsql";
        adminuser = "admin";
        adminpassFile = "/REPLACE/WITH/YOUR/PATH";
      };
      # Suggested by Nextcloud's health check.
      phpOptions."opcache.interned_strings_buffer" = "16";
    };
    # Nightly database backups.
    postgresqlBackup = {
      enable = true;
      startAt = "*-*-* 01:15:00";
    };
  };
}
```


**Unfinished**
