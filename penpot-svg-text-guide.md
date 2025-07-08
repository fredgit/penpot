# Contributing to Penpot: Solving the SVG Text Import Issue
 
 This guide provides comprehensive information on how to contribute to the Penpot project specifically to address the SVG text layer import limitations.
 
 ## Table of Contents
 
 1. [Understanding the Problem](#understanding-the-problem)
 2. [Technical Architecture Overview](#technical-architecture-overview)
 3. [Development Environment Setup](#development-environment-setup)
 4. [Key Areas for Contribution](#key-areas-for-contribution)
 5. [Implementation Strategy](#implementation-strategy)
 6. [Code Contribution Process](#code-contribution-process)
 7. [Testing and Validation](#testing-and-validation)
 8. [Community Engagement](#community-engagement)
 
 ## Understanding the Problem
 
 ### Current SVG Text Import Limitations
 
 **The Issue**: When importing SVG files containing `<text>` elements into Penpot:
 - Text elements are either not imported or become non-editable
 - Text styling, positioning, and formatting information is lost
 - Complex text features (multi-line, rich formatting) are not supported
 - Font references may not be preserved correctly
 
 **Root Cause**: Penpot's SVG parser lacks comprehensive support for:
 - SVG `<text>` element parsing
 - Text positioning attributes (`x`, `y`, `dx`, `dy`)
 - Text styling properties
 - Font family and size conversion
 - Multi-line text handling (`<tspan>` elements)
 
 ### Impact
 This limitation affects users migrating from other design tools and reduces Penpot's utility for SVG-based workflows.
 
 ## Technical Architecture Overview
 
 ### Penpot's Technology Stack
 
 ```
 Frontend (ClojureScript)
 ├── UI Components (React-based)
 ├── SVG Parser/Renderer
 ├── Design System
 └── State Management
 
 Backend (Clojure)
 ├── API Layer
 ├── File Processing
 ├── Database (PostgreSQL)
 └── Assets Management
 
 Common Libraries
 ├── Geometry/Math Utils
 ├── SVG Processing
 └── Data Validation
 ```
 
 ### Key Repositories
 
 1. **Main Repository**: `https://github.com/penpot/penpot`
    - Frontend: `/frontend` directory
    - Backend: `/backend` directory 
    - Common: `/common` directory
 
 2. **Documentation**: `https://help.penpot.app/technical-guide/`
 
 3. **Community Files**: `https://github.com/penpot/penpot-files`
 
 ## Development Environment Setup
 
 ### Prerequisites
 
 - **Docker & Docker Compose V2** (required)
 - **Git** for version control
 - **Node.js & npm** (specified in `.nvmrc`)
 - **Java 11+** for Clojure development
 - **Basic tmux knowledge** (development environment uses tmux)
 
 ### Quick Setup
 
 1. **Clone the Repository**
    ```bash
    git clone https://github.com/penpot/penpot.git
    cd penpot
    ```
 
 2. **Start Development Environment**
    ```bash
    ./manage.sh pull-devenv
    ./manage.sh run-devenv
    ```
 
 3. **Access Services**
    - Penpot App: `http://localhost:3449`
    - Storybook: `http://localhost:6006`
    - Email Testing: `http://localhost:1080`
 
 ### Development Environment Structure
 
 The tmux session provides:
 - **Window 0**: Gulp process (styles, fonts, templates)
 - **Window 1**: Shadow-cljs (frontend ClojureScript)
 - **Window 2**: Storybook server
 - **Window 3**: Exporter build process
 - **Window 4**: Backend REPL
 
 ### Frontend Development Setup
 
 1. **Connect to Frontend REPL**
    ```bash
    cd penpot/frontend
    npx shadow-cljs cljs-repl main
    ```
 
 2. **IDE Integration** (optional)
    - nREPL port: `3447`
    - Host: `localhost`
    - Execute: `(shadow/repl :main)` to connect
 
 ## Key Areas for Contribution
 
 ### 1. SVG Parser Enhancement (Priority: High)
 
 **Location**: `/frontend/src/app/common/svg/`
 
 **Current Implementation**: Basic SVG parsing without text support
 
 **Needed Improvements**:
 - Add `<text>` element parser
 - Implement `<tspan>` support for multi-line text
 - Handle text positioning attributes
 - Support text transformation matrices
 
 **Key Files**:
 - `parser.cljs` - Main SVG parsing logic
 - `svg.cljs` - SVG utilities and helpers
 
 ### 2. Text Conversion Logic (Priority: High)
 
 **Location**: `/frontend/src/app/main/data/workspace/svg_upload.cljs`
 
 **Current Gap**: SVG text elements are not converted to Penpot text objects
 
 **Implementation Needs**:
 ```clojure
 (defn parse-svg-text-element [text-element]
   "Convert SVG <text> element to Penpot text shape"
   {:content (extract-text-content text-element)
    :position (parse-text-position text-element)
    :typography (parse-text-styles text-element)
    :transform (parse-text-transform text-element)})
 ```
 
 ### 3. Typography System Integration (Priority: Medium)
 
 **Location**: `/frontend/src/app/common/types/typography.cljs`
 
 **Enhancement**: Map SVG text attributes to Penpot typography tokens
 
 **Mapping Required**:
 - `font-family` → Penpot font references
 - `font-size` → Penpot size system
 - `font-weight`, `font-style` → Penpot typography variants
 - `fill`, `stroke` → Penpot color system
 
 ### 4. Backend Processing (Priority: Medium)
 
 **Location**: `/backend/src/app/http/assets.clj`
 
 **Enhancement**: Server-side SVG text preprocessing
 
 **Features to Add**:
 - Font validation and substitution
 - Text measurement and positioning calculations
 - Large file processing optimization
 
 ## Implementation Strategy
 
 ### Phase 1: Basic Text Import
 
 **Goal**: Import simple SVG text elements as editable Penpot text
 
 **Tasks**:
 1. **Parser Extension**
    ```clojure
    ;; Add to svg/parser.cljs
    (defmethod parse-svg-element "text" [element]
      (let [content (.-textContent element)
            x (js/parseFloat (get-attribute element "x" "0"))
            y (js/parseFloat (get-attribute element "y" "0"))
            styles (parse-style-attribute element)]
        {:type :text
         :content content
         :position {:x x :y y}
         :typography (styles->typography styles)}))
    ```
 
 2. **Shape Creation**
    ```clojure
    ;; Add to workspace/svg_upload.cljs
    (defn create-text-shape [svg-text-data]
      (merge (default-text-shape)
             {:content (:content svg-text-data)
              :position-data (:position svg-text-data)
              :typography (:typography svg-text-data)}))
    ```
 
 3. **Typography Conversion**
    ```clojure
    ;; Add typography mapping utilities
    (defn svg-font-family->penpot [font-family]
      (or (get font-mapping font-family)
          default-font))
    ```
 
 ### Phase 2: Advanced Text Features
 
 **Goal**: Support complex text formatting and positioning
 
 **Tasks**:
 1. **Multi-line Text Support**
    - Parse `<tspan>` elements
    - Handle line breaks and spacing
    - Preserve relative positioning
 
 2. **Rich Text Formatting**
    - Support inline styling changes
    - Handle text decoration (underline, strike-through)
    - Preserve character spacing
 
 3. **Font Management**
    - Integrate with Penpot's font system
    - Handle missing font fallbacks
    - Support custom font imports
 
 ### Phase 3: Optimization and Polish
 
 **Goal**: Performance optimization and edge case handling
 
 **Tasks**:
 1. **Performance Improvements**
    - Lazy loading for large files
    - Optimize text rendering pipeline
    - Memory usage optimization
 
 2. **Error Handling**
    - Graceful degradation for unsupported features
    - User feedback for import limitations
    - Comprehensive logging
 
 ## Code Contribution Process
 
 ### 1. Setting Up Your Contribution
 
 1. **Fork the Repository**
    ```bash
    # Via GitHub UI or CLI
    gh repo fork penpot/penpot
    ```
 
 2. **Create Feature Branch**
    ```bash
    git checkout -b feature/svg-text-import
    ```
 
 3. **Follow Coding Standards**
    - Use existing code style (ClojureScript conventions)
    - Add comprehensive tests
    - Update documentation
 
 ### 2. Key Development Guidelines
 
 **ClojureScript Best Practices**:
 - Use meaningful function names
 - Keep functions small and focused
 - Add docstrings for public functions
 - Follow Penpot's naming conventions
 
 **Example Code Structure**:
 ```clojure
 (ns app.main.data.workspace.svg-text
   "SVG text element processing utilities"
   (:require [app.common.svg :as svg]
             [app.common.types.typography :as typography]))
 
 (defn parse-svg-text
   "Converts SVG text element to Penpot text shape data.
    
    Parameters:
    - element: DOM element representing SVG <text>
    
    Returns:
    - Map containing Penpot text shape data"
   [element]
   ;; Implementation here
   )
 ```
 
 ### 3. Testing Requirements
 
 **Unit Tests** (Required):
 ```clojure
 (ns app.test.svg-text-test
   (:require [cljs.test :refer-macros [deftest is testing]]
             [app.main.data.workspace.svg-text :as svg-text]))
 
 (deftest test-basic-text-parsing
   (testing "Simple text element parsing"
     (let [element (create-test-text-element "Hello World")
           result (svg-text/parse-svg-text element)]
       (is (= "Hello World" (:content result)))
       (is (number? (get-in result [:position :x]))))))
 ```
 
 **Integration Tests** (Recommended):
 - Test complete SVG file import workflow
 - Verify text editing functionality
 - Test typography system integration
 
 ### 4. Documentation Updates
 
 **Required Documentation**:
 - Update technical documentation
 - Add code comments and docstrings
 - Create user guide updates
 - Update changelog
 
 **Example Documentation Update**:
 ```markdown
 ## SVG Text Import
 
 Penpot now supports importing SVG files containing text elements:
 
 - **Simple Text**: Basic `<text>` elements with positioning and styling
 - **Multi-line Text**: Support for `<tspan>` elements and line breaks
 - **Typography**: Automatic mapping to Penpot's typography system
 - **Font Handling**: Intelligent font matching and fallbacks
 ```
 
 ### 5. Pull Request Process
 
 1. **Pre-PR Checklist**:
    - [ ] All tests pass
    - [ ] Code follows style guidelines
    - [ ] Documentation updated
    - [ ] Feature tested manually
    - [ ] No lint errors
 
 2. **PR Template** (follow this structure):
    ```markdown
    ## Description
    Implements SVG text element import functionality for Penpot.
    
    ## Changes Made
    - Added SVG text parser in `svg/parser.cljs`
    - Implemented text shape conversion
    - Updated typography mapping system
    
    ## Testing
    - Added unit tests for text parsing
    - Manually tested with various SVG files
    - Verified text editing functionality
    
    ## Related Issues
    Fixes #1706
    ```
 
 3. **Review Process**:
    - Maintainers will review code
    - Address feedback promptly
    - Be patient with the review cycle
    - Engage constructively with reviewers
 
 ## Testing and Validation
 
 ### Local Testing Setup
 
 1. **Create Test SVG Files**:
    ```svg
    <!-- Simple text test -->
    <svg xmlns="http://www.w3.org/2000/svg" width="200" height="100">
      <text x="10" y="30" font-family="Arial" font-size="16" fill="black">
        Hello Penpot!
      </text>
    </svg>
    
    <!-- Complex text test -->
    <svg xmlns="http://www.w3.org/2000/svg" width="300" height="150">
      <text x="10" y="30" font-family="Arial" font-size="16">
        <tspan x="10" y="30" fill="red">Line 1</tspan>
        <tspan x="10" y="50" fill="blue">Line 2</tspan>
      </text>
    </svg>
    ```
 
 2. **Test Import Process**:
    - Import SVG through Penpot UI
    - Verify text appears as editable text layers
    - Check typography preservation
    - Test text editing functionality
 
 3. **Performance Testing**:
    - Test with large SVG files
    - Monitor memory usage
    - Check import speed
 
 ### Automated Testing
 
 **Frontend Tests**:
 ```bash
 # Run frontend tests
 cd frontend
 npm test
 ```
 
 **Backend Tests**:
 ```bash
 # Run backend tests
 cd backend
 lein test
 ```
 
 **Integration Tests**:
 ```bash
 # Run full test suite
 ./manage.sh run-tests
 ```
 
 ## Community Engagement
 
 ### Ways to Contribute Beyond Code
 
 1. **Community Discussion**:
    - Join [Penpot Community](https://community.penpot.app/)
    - Participate in SVG import discussions
    - Share your use cases and requirements
 
 2. **Issue Reporting**:
    - Test SVG import functionality
    - Report bugs with detailed reproduction steps
    - Suggest feature improvements
 
 3. **Documentation**:
    - Improve user guides
    - Create tutorials
    - Translate documentation
 
 4. **Testing and Feedback**:
    - Test development builds
    - Provide feedback on new features
    - Help with beta testing
 
 ### Getting Help
 
 **Development Support**:
 - **Discord/Matrix**: Real-time development chat
 - **GitHub Discussions**: Technical questions
 - **Community Forum**: General questions and feedback
 
 **Technical Questions**:
 - Review existing issues and PRs
 - Check the technical documentation
 - Ask specific questions in discussions
 
 **Contribution Recognition**:
 - All contributors are recognized in release notes
 - Significant contributions highlighted in community updates
 - Opportunity to become a maintainer
 
 ### Long-term Collaboration
 
 **Becoming a Core Contributor**:
 1. Start with small contributions
 2. Build relationship with community
 3. Take on larger features
 4. Help with code reviews
 5. Mentor new contributors
 
 **Areas for Ongoing Development**:
 - SVG import/export improvements
 - Performance optimizations
 - Cross-platform compatibility
 - Plugin system development
 
 ## Additional Resources
 
 ### Learning Resources
 
 **ClojureScript**:
 - [ClojureScript Guide](https://clojurescript.org/guides/quick-start)
 - [Re-frame Documentation](http://day8.github.io/re-frame/)
 - [Shadow-cljs User Guide](https://shadow-cljs.github.io/docs/UsersGuide.html)
 
 **SVG Specifications**:
 - [SVG 1.1 Specification](https://www.w3.org/TR/SVG11/)
 - [SVG Text Module](https://www.w3.org/TR/SVG11/text.html)
 - [MDN SVG Documentation](https://developer.mozilla.org/en-US/docs/Web/SVG)
 
 **Penpot Specific**:
 - [Technical Guide](https://help.penpot.app/technical-guide/)
 - [Architecture Documentation](https://help.penpot.app/technical-guide/developer/architecture/)
 - [Contributing Guide](https://github.com/penpot/penpot/blob/develop/CONTRIBUTING.md)
 
 ### Tools and Utilities
 
 **Development Tools**:
 - [Calva](https://calva.io/) - VS Code ClojureScript support
 - [CIDER](https://docs.cider.mx/) - Emacs ClojureScript REPL
 - [Cursive](https://cursive-ide.com/) - IntelliJ ClojureScript plugin
 
 **SVG Tools**:
 - [SVG Path Visualizer](https://svg-path-visualizer.netlify.app/)
 - [SVGO](https://github.com/svg/svgo) - SVG optimization
 - [SVG Editor](https://boxy-svg.com/) - Online SVG editor
 
 ## Conclusion
 
 Contributing to Penpot's SVG text import functionality is a valuable way to improve the platform for thousands of users. This feature would significantly enhance Penpot's usability and help establish it as a more complete Figma alternative.
 
 Start small, engage with the community, and gradually take on larger challenges. Your contributions will help shape the future of open-source design tools.
 
 **Ready to contribute? Start here**:
 1. Set up your development environment
 2. Join the Penpot community discussions
 3. Pick up a beginner-friendly issue
 4. Make your first contribution!

