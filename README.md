# tips-and-tricks
Random tips + tricks

## Git tricks
- Count number of author commits
```
git log --author="_Your_Name_Here_" --pretty=tformat: --numstat \
| gawk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s removed lines: %s total lines: %s\n", add, subs, loc }' -
```

- List of git commit counts by author
```
git shortlog -s
```

- Count git commits from author
```
git rev-list HEAD --author="Shing Lyu" --count 
```

- Count git commits from author since date
```
git rev-list HEAD --author="Shing Lu" --count --since="23/05/2019"
```

- Repo analyzer
```
#!/bin/bash

# Colors for pretty output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
PURPLE='\033[0;35m'
CYAN='\033[0;36m'
WHITE='\033[1;37m'
NC='\033[0m' # No Color

# Function to print a separator line
print_separator() {
    echo -e "${CYAN}$(printf '=%.0s' {1..80})${NC}"
}

# Function to print a header
print_header() {
    echo -e "${WHITE}$1${NC}"
    print_separator
}

# Function to analyze a single repository
analyze_repo() {
    local repo_name=$1
    
    if [ ! -d "$repo_name" ]; then
        echo -e "${RED}Repository $repo_name not found!${NC}"
        return 1
    fi
    
    cd "$repo_name"
    
    print_header "📊 ANALYSIS FOR: $repo_name"
    
    # Get total commits and authors
    total_commits=$(git rev-list --all --count)
    total_authors=$(git log --format='%aN' | sort -u | wc -l)
    
    echo -e "${GREEN}Total Commits: $total_commits${NC}"
    echo -e "${GREEN}Total Authors: $total_authors${NC}"
    echo ""
    
    # 1. Commit count per author (sorted by commits)
    echo -e "${YELLOW}📝 COMMITS PER AUTHOR:${NC}"
    git shortlog -sn --all | head -20 | while read commits author; do
        printf "${BLUE}%4d${NC} commits - ${WHITE}%s${NC}\n" "$commits" "$author"
    done
    echo ""
    
    # 2. Lines added/deleted per author
    echo -e "${YELLOW}📈 LINES OF CODE TOUCHED PER AUTHOR:${NC}"
    git log --format='%aN' --numstat | awk '
    NF==3 {plus+=$1; minus+=$2}
    NF==1 {
        if(author != "") {
            printf "%s: +%d -%d (net: %+d)\n", author, plus, minus, plus-minus
        }
        author=$0; plus=0; minus=0
    }
    END {
        if(author != "") {
            printf "%s: +%d -%d (net: %+d)\n", author, plus, minus, plus-minus
        }
    }' | sort -k4 -nr | head -20 | while IFS=': ' read -r author stats; do
        printf "${PURPLE}%-25s${NC} ${GREEN}%s${NC}\n" "$author" "$stats"
    done
    echo ""
    
    # 3. Recent activity (last 10 commits)
    echo -e "${YELLOW}🕐 RECENT ACTIVITY (Last 10 commits):${NC}"
    git log --oneline --format="%C(yellow)%h%C(reset) %C(blue)%an%C(reset) %C(green)(%ar)%C(reset) %s" -10
    echo ""
    
    # 4. File type statistics
    echo -e "${YELLOW}📁 FILE TYPES IN REPOSITORY:${NC}"
    git ls-files | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10 | while read count ext; do
        printf "${CYAN}%4d${NC} .%-10s files\n" "$count" "$ext"
    done
    echo ""
    
    cd ..
}

# Main execution
print_header "🚀 GIT REPOSITORY ANALYSIS TOOL"
echo -e "${WHITE}Analyzing repositories: team404-sql-agent & team-404-sql-agent-frontend${NC}"
echo ""

# Analyze both repositories
analyze_repo "team404-sql-agent"
echo ""
analyze_repo "team-404-sql-agent-frontend"

# Combined analysis
print_header "🔀 COMBINED ANALYSIS"

# Create temporary files for combined data
temp_commits=$(mktemp)
temp_lines=$(mktemp)

for repo in "team404-sql-agent" "team-404-sql-agent-frontend"; do
    if [ -d "$repo" ]; then
        cd "$repo"
        git shortlog -sn --all >> "$temp_commits"
        git log --format='%aN' --numstat | awk '
        NF==3 {plus+=$1; minus+=$2}
        NF==1 {
            if(author != "") {
                printf "%s %d %d\n", author, plus, minus
            }
            author=$0; plus=0; minus=0
        }
        END {
            if(author != "") {
                printf "%s %d %d\n", author, plus, minus
            }
        }' >> "$temp_lines"
        cd ..
    fi
done

# Combined commit analysis
echo -e "${YELLOW}📊 COMBINED COMMITS BY AUTHOR:${NC}"
awk '{commits[$0] += $1; $1=""; author=substr($0,2)} END {for(a in commits) print commits[a], a}' "$temp_commits" | sort -nr | head -20 | while read commits author; do
    printf "${BLUE}%4d${NC} commits - ${WHITE}%s${NC}\n" "$commits" "$author"
done
echo ""

# Combined lines analysis
echo -e "${YELLOW}📈 COMBINED LINES TOUCHED BY AUTHOR:${NC}"
awk '{
    author=$1; 
    for(i=2; i<NF; i++) author=author " " $i;
    plus[author]+=$NF-1; minus[author]+=$NF
} END {
    for(a in plus) printf "%s: +%d -%d (net: %+d)\n", a, plus[a], minus[a], plus[a]-minus[a]
}' "$temp_lines" | sort -k4 -nr | head -20 | while IFS=': ' read -r author stats; do
    printf "${PURPLE}%-25s${NC} ${GREEN}%s${NC}\n" "$author" "$stats"
done

# Cleanup
rm "$temp_commits" "$temp_lines"

print_separator
echo -e "${WHITE}Analysis complete! 🎉${NC}"
```
