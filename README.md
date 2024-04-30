# Bash Git Prompt
BGP_BRANCH_SAFE=("develop")
BGP_BRANCH_UNSAFE=("master")
BGP_USER_UNSAFE=("root" "prod" "git")

function in_array() {
  local e
  for e in "${@:2}"; do [[ "$e" == "$1" ]] && return 0; done
  return 1
}

function bgp_prompt() {

  # Define colors
  local RED="\[\e[31m\]"
  local YELLOW="\[\e[33m\]"
  local GREEN="\[\e[32m\]"
  local BLUE="\[\e[34m\]"
  local RESET="\[$(tput sgr0)\]"

  # Get username and host
  local USER=$(whoami)
  local USER_PS="${GREEN}\u${RESET}"
  if in_array "${USER}" "${BGP_USER_UNSAFE[@]}"; then
    USER_PS="${RED}\u@\h${RESET}"
  fi

  # Get git status
  local GIT_STATUS=$(git status --porcelain --ignore-submodules 2> /dev/null | wc -l)
  local GIT_STATUS_PS="$GREEN=$RESET"
  if [[ "0" != "$GIT_STATUS" ]]; then
    GIT_STATUS_PS="$RED~$GIT_STATUS$RESET"
  fi

  # Get git branch
  local GIT_BRANCH=$(git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/\1/')
  local GIT_BRANCH_PS=" $BLUE($GIT_BRANCH)$RESET$GIT_STATUS_PS"
  if in_array "${GIT_BRANCH}" "${BGP_BRANCH_SAFE[@]}"; then
    GIT_BRANCH_PS=" $GREEN($GIT_BRANCH)$RESET$GIT_STATUS_PS"
  elif in_array "${GIT_BRANCH}" "${BGP_BRANCH_UNSAFE[@]}"; then
    GIT_BRANCH_PS=" $RED($GIT_BRANCH)$RESET$GIT_STATUS_PS"
  elif [[ "" == "$GIT_BRANCH" ]]; then
    GIT_BRANCH_PS=""
  fi

  PS1="${USER_PS} ${YELLOW}\w${RESET}${GIT_BRANCH_PS} \\$ "

}

PROMPT_COMMAND=bgp_prompt
