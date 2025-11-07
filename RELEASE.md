# CodGuard Release Guide

This repository keeps the development sources without committing `vendor/`. A GitHub Action builds the WordPress-ready ZIP whenever a semantic tag is pushed. This document explains the release process for maintainers and how to verify the automation locally.

## 1. Prerequisites

- PHP 8.1+
- Composer 2
- Zip utility (`zip` on macOS/Linux, PowerShell `Compress-Archive` on Windows)
- Git access to the repository with permission to push tags

## 2. Release Workflow Overview

The action defined in `.github/workflows/release.yml` triggers on tags starting with `v` (for example, `v2.1.11`). It performs the following steps:

1. Check out the tagged commit.
2. Install Composer dependencies for production: `composer install --no-dev --prefer-dist --no-progress --no-interaction --optimize-autoloader`.
3. Copy the repository contents into `build/codguard/`, excluding development artifacts (`.git`, `.github`, `var`, phpstan configuration files, etc.).
4. Create `build/codguard.zip` containing the plugin directory.
5. Upload the archive as an asset to the GitHub release that is automatically created for the tag.

End users can download `codguard.zip`, unzip it, and install the plugin by uploading the `codguard/` directory to `wp-content/plugins/` or by using the WordPress admin plugin uploader.

## 3. Creating a Release

1. Ensure the main branch is clean and all changes are reviewed.
2. Update metadata (version constants, changelog) as needed.
3. Commit your changes.
4. Create and push a tag:

   ```bash
   git tag v2.1.11
   git push origin v2.1.11
   ```

5. Wait for the GitHub Action to finish. A release named after the tag will appear with `codguard.zip` attached.

## 4. Manual Verification (Optional)

To emulate the workflow locally:

```bash
composer install --no-dev --prefer-dist --no-progress --no-interaction --optimize-autoloader
mkdir -p build
rsync -av \
  --exclude='.git' \
  --exclude='.github' \
  --exclude='build' \
  --exclude='var' \
  --exclude='phpstan.neon.dist' \
  --exclude='phpstan-bootstrap.php' \
  --exclude='.gitignore' \
  ./ build/codguard
cd build
zip -r codguard.zip codguard
```

After verification, restore development dependencies and clean up:

```bash
cd ..
rm -rf build
composer install
```

## 5. Cleaning Development Dependencies

The `.gitignore` file excludes `build/` and temporary vendor packages (`phpstan`, stubs, coding standards). Developers should keep those locally by running `composer install` (with dev dependencies). They are intentionally omitted from the production ZIP.

## 6. Troubleshooting

- **Missing ZIP on release** – Verify that the tag name starts with `v` and that the workflow finished successfully (check the Actions tab).
- **Composer install failures** – Ensure your PHP version meets the constraints in `composer.json` and that Composer can reach Packagist.
- **WordPress installation issues** – Confirm that the deployed ZIP still contains `codguard/composer.json`, `codguard/vendor/`, and the plugin bootstrap `codguard.php`.

For further automation adjustments, edit `.github/workflows/release.yml` and keep this guide in sync.

