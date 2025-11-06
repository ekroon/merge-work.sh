#!/usr/bin/env -S mise x gh -- bash

# Check if gh CLI is installed
if ! command -v gh &>/dev/null; then
  echo "Error: GitHub CLI (gh) is not installed. Please install it first."
  echo "Visit: https://cli.github.com/"
  exit 1
fi

# Ensure we're logged in
if ! gh auth status &>/dev/null; then
  echo "Error: Not logged in to GitHub CLI. Please run 'gh auth login' first."
  exit 1
fi

# Function to process a single PR
process_pr() {
  pr_number=$1
  pr_title=$2
  is_draft=$3

  echo
  echo "==============================================================================="
  echo "PR #$pr_number: $pr_title"
  echo "==============================================================================="

  if [ "$is_draft" = "true" ]; then
    echo "STATUS: Draft PR"
    while true; do
      echo -n "Would you like to mark PR #$pr_number \"$pr_title\" as ready for review? (y/n): "
      read -r response < /dev/tty
      if [[ $response =~ ^[yn]$ ]]; then
        mark_ready=$response
        break
      fi
      echo "Please answer y or n"
    done

    if [[ $mark_ready == "y" ]]; then
      echo "Marking PR as ready for review..."
      gh pr ready "$pr_number" -R github/issues-platform
    else
      echo "Skipping draft PR"
      return
    fi
  fi

  while true; do
    echo -n "Would you like to merge PR #$pr_number \"$pr_title\"? (y/n): "
    read -r response < /dev/tty
    if [[ $response =~ ^[yn]$ ]]; then
      confirm=$response
      break
    fi
    echo "Please answer y or n"
  done

  if [[ $confirm == "y" ]]; then
    echo "Approving and merging PR..."
    gh pr review "$pr_number" -R github/issues-platform --approve
    gh pr merge "$pr_number" -R github/issues-platform --auto --merge
    echo "Merge complete for \"$pr_title\""
  else
    echo "Skipping PR"
  fi
}

# Get all PRs assigned to the current user
echo "Fetching your assigned PRs from github/issues-platform..."
prs=$(gh pr list -R github/issues-platform --assignee "@me" --json number,title,isDraft)

if [ -z "$prs" ]; then
  echo "No PRs found assigned to you in github/issues-platform"
  exit 0
fi

# Display summary of PRs first
echo
echo "Found the following PRs assigned to you:"
echo "==============================================================================="
echo "$prs" | jq -r '.[] | "PR #\(.number): \(.title) \(if .isDraft then "(Draft)" else "" end)"'
echo "==============================================================================="
echo

# Ask if user wants to proceed
while true; do
  echo -n "Would you like to process these PRs? (y/n): "
  read -r response < /dev/tty
  if [[ $response =~ ^[yn]$ ]]; then
    proceed=$response
    break
  fi
  echo "Please answer y or n"
done

if [[ $proceed != "y" ]]; then
  echo "Operation cancelled"
  exit 0
fi

# Process each PR
while IFS= read -r json_line; do
  number=$(echo "$json_line" | jq -r '.number')
  title=$(echo "$json_line" | jq -r '.title')
  is_draft=$(echo "$json_line" | jq -r '.isDraft')

  process_pr "$number" "$title" "$is_draft"
done < <(echo "$prs" | jq -c '.[]')

echo
echo "PR processing complete!"
echo "You can view all your assigned PRs at: https://github.com/github/issues-platform/pulls?q=is:pr+is:open+assignee:@me"
